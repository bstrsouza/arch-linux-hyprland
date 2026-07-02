# Configuração de Múltiplos Monitores

---

## Contexto

O Hyprland permite configurar quantos monitores forem necessários via `hl.monitor()`, cada um com sua própria resolução, posição e escala. Este guia documenta a configuração usada neste ambiente: o display interno do laptop (Samsung 960XFH) mais 2 monitores externos conectados via dock/hub USB-C.

**Layout físico:** os dois monitores externos ficam lado a lado, acima do laptop, que fica centralizado na mesa embaixo deles.

| Monitor | Papel | Conector |
|---|---|---|
| Samsung 960XFH | Display interno do laptop | `eDP-1` |
| Externo esquerdo | Monitor 1920x1080 | `DP-5` |
| Externo direito | Monitor 1920x1080 | `DP-6` |

---

## Identificar os monitores conectados

```bash
hyprctl monitors all
```

Mostra o nome do conector (`eDP-1`, `DP-5`, `DP-6`...), modo ativo, posição no layout e descrição (fabricante/modelo).

> **Atenção:** monitores conectados via dock/hub USB-C às vezes reportam um EDID genérico idêntico (ex: `LTM LCD 0x01010101` para os dois externos deste setup), então a `description` não serve para diferenciar qual é qual fisicamente.

**Truque para descobrir qual conector é qual monitor físico:** foque cada monitor e abra um terminal colorido nele.

```bash
hyprctl dispatch 'hl.dsp.focus({monitor="DP-5"})'
hyprctl dispatch 'hl.dsp.exec_cmd("kitty -o background=#ff0000 --title ID sh -c \"echo DP-5; sleep 8\"")'
```

Repita trocando `DP-5` por `DP-6` (e a cor) para o outro monitor. O terminal aparece na tela física correspondente por 8 segundos.

---

## Configurar posição, resolução e escala

Com os dois externos conectados, a posição de cada um é fixa e a do laptop é calculada para ficar centralizado embaixo dos dois:

| Monitor | Resolução | Posição | Escala |
|---|---|---|---|
| `DP-5` (externo esquerdo) | 1920x1080@60 | `0x0` | 1 |
| `DP-6` (externo direito) | 1920x1080@60 | `1920x0` | 1 |
| `eDP-1` (laptop, embaixo, centralizado) | 2880x1800@60 | `960x1080` | 1.5 |

`position` usa a notação `XxY` em pixels, a partir do canto superior esquerdo do layout virtual. Para centralizar o laptop embaixo dos dois externos:

```
largura_topo   = 1920 (DP-5) + 1920 (DP-6) = 3840
largura_laptop = 2880 / 1.5 (escala) = 1920   -- tamanho lógico
x_laptop       = (3840 - 1920) / 2 = 960
y_laptop       = 1080                          -- altura dos monitores de cima
```

---

## Configuração condicional — funciona com 0, 1 ou 2 externos, com hotplug automático

Esse cálculo só dá certo se os **dois** externos estiverem conectados. Como este laptop às vezes é usado sozinho, às vezes com 1 monitor externo e às vezes com 2, a posição do `eDP-1` é recalculada usando `hl.get_monitor(nome)` (retorna `nil` se o monitor não estiver conectado), dentro de uma função reaplicada tanto no carregamento da config quanto a cada plug/unplug, via `hl.on("monitor.added"/"monitor.removed", ...)`:

```lua
local EXTERNAL_WIDTH = 1920 -- DP-5 / DP-6, 1920x1080 sem escala
local LAPTOP_WIDTH    = 1920 -- 2880 / escala 1.5 (tamanho lógico do eDP-1)

local function applyMonitorLayout()
    local hasDP5 = hl.get_monitor("DP-5") ~= nil
    local hasDP6 = hl.get_monitor("DP-6") ~= nil

    if hasDP5 then
        hl.monitor({output = "DP-5", mode = "1920x1080@60", position = "0x0", scale = "1"})
    end

    if hasDP6 then
        -- Se DP-5 também estiver conectado, DP-6 vai ao lado (1920x0).
        -- Se DP-6 for o único externo, ele assume a posição 0x0.
        local dp6X = hasDP5 and EXTERNAL_WIDTH or 0
        hl.monitor({output = "DP-6", mode = "1920x1080@60", position = dp6X .. "x0", scale = "1"})
    end

    local topRowWidth = (hasDP5 and EXTERNAL_WIDTH or 0) + (hasDP6 and EXTERNAL_WIDTH or 0)
    local laptopX = topRowWidth > 0 and math.floor((topRowWidth - LAPTOP_WIDTH) / 2) or 0
    local laptopY = topRowWidth > 0 and 1080 or 0

    hl.monitor({output = "eDP-1", mode = "2880x1800@60", position = laptopX .. "x" .. laptopY, scale = "1.5"})

    -- Ver seção "Waybar/wallpaper grudam..." abaixo
    hl.dispatch(hl.dsp.force_renderer_reload())
end

applyMonitorLayout()

hl.on("monitor.added",   applyMonitorLayout)
hl.on("monitor.removed", applyMonitorLayout)
```

Resultado em cada cenário (validado simulando a fórmula com `lua5.4` e, para o caso dos 2 externos, comparando com `hyprctl monitors all` ao vivo):

| Cenário | `DP-5` | `DP-6` | `eDP-1` |
|---|---|---|---|
| Só o laptop | — | — | `0x0` |
| Laptop + só `DP-5` | `0x0` | — | `0x1080` (centralizado sob o único externo) |
| Laptop + só `DP-6` | — | `0x0` (realocado) | `0x1080` (centralizado sob o único externo) |
| Laptop + `DP-5` + `DP-6` | `0x0` | `1920x0` | `960x1080` (centralizado sob os dois) |

Repare que `hl.get_monitor()` é chamado **fora** de qualquer `hl.monitor()`, e que cada monitor ainda recebe sua própria chamada separada — o bug abaixo continua valendo mesmo dentro da função.

**Sobre o hotplug:** `hl.on("monitor.removed", ...)` dispara com o monitor já removido do estado interno (`hl.get_monitor()`/`hl.get_monitors()` já refletem a topologia pós-remoção), então `applyMonitorLayout()` sempre calcula com dados atuais. Testado ao vivo criando/removendo um monitor virtual (sem precisar desconectar hardware de verdade):

```bash
hyprctl output create headless   # dispara monitor.added
hyprctl output remove HEADLESS-N # dispara monitor.removed
```

> Curiosidade: `monitor.removed` disparou 3x seguidas para uma única remoção nos testes (mesmo resultado nas 3 vezes). Inofensivo, já que `applyMonitorLayout()` é idempotente — só custa 2 chamadas extras à toa.

---

## ⚠️ Bug — Waybar/wallpaper grudam na posição antiga ao remover um monitor

Reposicionar `eDP-1` via `hl.monitor()` dentro do handler de `monitor.removed` funciona (o monitor em si vai para a posição nova — confirmado com `hyprctl monitors`), mas as layer-shell surfaces já abertas nele (Waybar, o daemon de wallpaper `awww-daemon`) **não acompanham**: `hyprctl layers` continua mostrando o `xywh` antigo, indefinidamente (testei parado 10s sem nada mudar). Isso deixa a Waybar (e o wallpaper) visualmente fora do lugar depois de desconectar um monitor.

**Causa:** ao remover um monitor, o Hyprland não força as layer-shell surfaces do monitor sobrevivente a refazer o layout quando a posição dele muda como efeito colateral da remoção.

**Solução:** disparar o dispatcher `force_renderer_reload` logo depois de reaplicar os `hl.monitor()` — por isso ele está dentro de `applyMonitorLayout()` lá em cima, não só nos `hl.on(...)`:

```lua
hl.dispatch(hl.dsp.force_renderer_reload())
```

Um `hyprctl reload` completo também resolve (é o que eu rodava manualmente antes de automatizar isso), mas `force_renderer_reload()` é mais leve — não reprocessa a config inteira, só força o redraw/relayout.

**Reprodução determinística usada para validar** (sem precisar plugar/desplugar hardware de verdade): registrei um handler temporário que reposiciona o `eDP-1` para `500x500` em `monitor.added` e de volta pra `0x0` em `monitor.removed`, e disparei os dois eventos com `hyprctl output create/remove headless`. Sem o `force_renderer_reload()`, `hyprctl layers` ficava preso em `500 500`; com ele, ficava em `0 0` imediatamente.

Ver também [TROUBLESHOOTING.md](../TROUBLESHOOTING.md).

---

## ⚠️ Bug — múltiplos monitores na mesma chamada `hl.monitor()`

Passar várias tabelas numa única chamada **não funciona** — apenas a `position` da primeira entrada é respeitada; as demais caem em `"auto"`, ignorando a posição explícita (e podem até alterar `scale`/modo de vídeo do monitor afetado).

```lua
-- Errado — posição de DP-6 e eDP-1 é ignorada
hl.monitor(
    {output = "DP-5",  mode = "1920x1080@60", position = "0x0",      scale = "1"},
    {output = "DP-6",  mode = "1920x1080@60", position = "1920x0",   scale = "1"},
    {output = "eDP-1", mode = "2880x1800@60", position = "960x1080", scale = "1.5"}
)

-- Correto — uma chamada hl.monitor() por monitor
hl.monitor({output = "DP-5",  mode = "1920x1080@60", position = "0x0",      scale = "1"})
hl.monitor({output = "DP-6",  mode = "1920x1080@60", position = "1920x0",   scale = "1"})
hl.monitor({output = "eDP-1", mode = "2880x1800@60", position = "960x1080", scale = "1.5"})
```

Ver também [TROUBLESHOOTING.md](../TROUBLESHOOTING.md).

---

## Mover workspaces entre monitores

Dispatchers em `hl.dsp.workspace`, descobertos via o stub oficial `/usr/share/hypr/stubs/hl.meta.lua` (não documentados na wiki no momento da escrita):

```lua
-- Move o workspace 4 para o monitor DP-6 (e já foca nele)
hl.dsp.workspace.move({ workspace = 4, monitor = "DP-6" })

-- Sem "workspace", move o workspace ATIVO no momento
hl.dsp.workspace.move({ monitor = "DP-6" })

-- Troca os workspaces ativos entre dois monitores
hl.dsp.workspace.swap_monitors({ monitor1 = "DP-5", monitor2 = "DP-6" })
```

`monitor` aceita tanto o nome do conector (`"DP-6"`) quanto um deslocamento relativo (`"+1"` / `"-1"`), que navega pela ordem interna dos IDs dos monitores — não necessariamente pela posição física esquerda/direita.

### Keybind configurado

Em `~/.config/hypr/hyprland.lua`:

```lua
hl.bind(mainMod .. " + SHIFT + right", hl.dsp.workspace.move({ monitor = "+1" }))
hl.bind(mainMod .. " + SHIFT + left",  hl.dsp.workspace.move({ monitor = "-1" }))
```

| Tecla | Ação |
|---|---|
| `Super + Shift + →` | Move o workspace atual para o próximo monitor |
| `Super + Shift + ←` | Move o workspace atual para o monitor anterior |

---

## Comandos úteis

```bash
# Ver monitores ativos: modo, posição, escala
hyprctl monitors all

# Ver em qual monitor cada workspace está
hyprctl workspaces

# Testar uma mudança de posição/monitor sem editar o arquivo
hyprctl dispatch 'hl.dsp.workspace.move({workspace=N, monitor="NOME"})'
```
