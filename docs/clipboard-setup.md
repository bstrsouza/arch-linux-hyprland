# Configuração do Clipboard — cliphist + wl-clipboard + wofi + wtype

---

## Contexto

O Wayland não tem um gerenciador de clipboard persistente por padrão — ao fechar um app, o conteúdo copiado nele é perdido. A solução é um daemon que monitora o clipboard e armazena o histórico.

**Componentes:**

| Pacote | Função |
|---|---|
| `wl-clipboard` | Implementa o clipboard do Wayland (`wl-copy` / `wl-paste`) |
| `cliphist` | Armazena e gerencia o histórico de itens copiados |
| `wofi` | Interface visual para selecionar um item do histórico |
| `wtype` | Digita o item selecionado diretamente na janela ativa |

---

## Instalação

```bash
sudo pacman -S wl-clipboard cliphist wofi wtype
```

---

## Autostart

Em `~/.config/hypr/hyprland.lua`, os watchers são iniciados com o Hyprland:

```lua
hl.exec_cmd("wl-paste --type text --watch cliphist store")
hl.exec_cmd("wl-paste --type image --watch cliphist store")
```

Monitoram o clipboard continuamente e salvam cada item copiado no banco do cliphist (`~/.cache/cliphist/db`).

---

## Atalho

Em `~/.config/hypr/hyprland.lua`:

```lua
hl.bind(mainMod .. " + V", hl.dsp.exec_cmd("sh -c 'cliphist list | wofi --dmenu | cliphist decode | wtype -'"))
```

| Tecla | Ação |
|---|---|
| `Super + V` | Abre o histórico, selecione um item e ele é digitado automaticamente |

**Fluxo:**
1. `cliphist list` — lista o histórico
2. `wofi --dmenu` — exibe a lista para seleção
3. `cliphist decode` — decodifica o item selecionado
4. `wtype -` — digita o conteúdo direto na janela ativa

---

## Comandos úteis

```bash
# Listar histórico no terminal
cliphist list

# Limpar todo o histórico
cliphist wipe
```
