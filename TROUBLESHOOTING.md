# Troubleshooting

Problemas encontrados durante a configuração deste ambiente e como foram resolvidos.

---

## Hyprland 0.55 — mudança no protocolo de dispatch

**Sintoma:** Comandos como `hyprctl dispatch exec blueman-manager` retornam erro de sintaxe Lua.

**Causa:** O Hyprland 0.55 com config Lua descontinuou o formato antigo de dispatch via socket.

**Solução:** Chamar os aplicativos diretamente, sem o wrapper `hyprctl dispatch exec`.

```jsonc
// Errado
"on-click": "hyprctl dispatch exec blueman-manager"

// Correto
"on-click": "blueman-manager"
```

Afeta todos os `on-click` da Waybar e qualquer lugar que use `hyprctl dispatch exec`.

---

## Waybar — clique nos workspaces não funciona

**Sintoma:** Clicar nos workspaces na Waybar não troca de workspace.

**Causa:** O módulo `hyprland/workspaces` da Waybar usa internamente `dispatch workspace X`
(formato antigo de socket), que foi descontinuado no Hyprland 0.55.

**Status:** Pendente de atualização da Waybar. A troca por teclado (`Super + 1..9`) funciona normalmente.

**Workaround:** Nenhum disponível sem modificar o código-fonte da Waybar.

---

## SDDM — tema não carrega após instalação

**Sintoma:** O SDDM continua com o tema padrão mesmo após configurar `/etc/sddm.conf`.

**Causa:** Nome do tema com capitalização errada. O SDDM diferencia maiúsculas de minúsculas.

**Solução:** O nome em `Current=` deve ser idêntico ao nome do diretório em `/usr/share/sddm/themes/`.

```bash
# Verificar nome exato
ls /usr/share/sddm/themes/

# Correto
Current=catppuccin-macchiato-blue

# Errado
Current=Catppuccin-Macchiato-Blue
Current=catppuccin_macchiato_blue
```

---

## hyprpaper — incompatível com Hyprland 0.55

**Sintoma:** O hyprpaper 0.8.4 falha ao iniciar com Hyprland 0.55.

**Causa:** Incompatibilidade de versões entre hyprpaper 0.8.4 e Hyprland 0.55.

**Solução:** Usar o `swww` (pacote `swww`, binários `awww-daemon` e `awww`) como substituto.

```bash
sudo pacman -S swww

# Iniciar daemon
awww-daemon

# Definir wallpaper
awww img /caminho/para/imagem.png
```

> Atenção: o pacote se chama `swww` mas os binários instalados são `awww-daemon` e `awww`.

---

## swww — comando não encontrado

**Sintoma:** `swww-daemon: command not found` ou `swww: command not found`.

**Causa:** O pacote `swww` instala os binários com nomes diferentes: `awww-daemon` e `awww`.

**Solução:** Usar os nomes corretos:

```bash
awww-daemon   # em vez de swww-daemon
awww img      # em vez de swww img
```

---

## Window rule — erro no campo `move`

**Sintoma:** Hyprland reporta erro: `field 'move': expression vec2 requires two expressions separated by whitespace`

**Causa:** O campo `move = "center"` é inválido — `move` requer coordenadas `vec2`, não a string `"center"`.

**Solução:** Usar o campo `center = true` para centralizar janelas flutuantes.

```lua
-- Errado
hl.window_rule({ move = "center", float = true })

-- Correto
hl.window_rule({ center = true, float = true })
```

---

## wlogout — tela branca ou botões não funcionam

**Sintoma:** O wlogout abre com tela branca ou os botões não executam as ações.

**Causa:** CSS customizado com propriedades inválidas interfere na renderização e nos eventos de clique.

**Solução:** Manter o CSS mínimo. Evitar sobrescrever propriedades que afetem eventos de input.
O layout (`~/.config/wlogout/layout`) pode ser customizado sem problemas.

---

## Waybar — logs aparecendo no terminal

**Sintoma:** Mensagens de log da Waybar aparecem no terminal onde o Hyprland foi iniciado.

**Causa:** A Waybar envia saída para stdout/stderr por padrão.

**Solução:** Redirecionar a saída ao iniciar:

```lua
hl.exec_cmd("waybar > /dev/null 2>&1")
```

---

## .zshrc — prompt Starship não carrega

**Sintoma:** O prompt do Starship não aparece ou o PATH não é carregado corretamente.

**Causa:** `eval "$(starship init zsh)"` e `export PATH=...` escritos na mesma linha sem separação.

**Solução:** Garantir que cada comando esteja em sua própria linha no `~/.zshrc`:

```zsh
# Errado
eval "$(starship init zsh)"export PATH="$HOME/.local/bin:$PATH"

# Correto
export PATH="$HOME/.local/bin:$PATH"
eval "$(starship init zsh)"
```

---

## hyprlauncher --dmenu — seleção não funciona

**Sintoma:** Ao usar `cliphist list | hyprlauncher --dmenu | cliphist decode | wl-copy`,
o item selecionado não é copiado para o clipboard.

**Causa:** O `hyprlauncher --dmenu` não envia a seleção para o stdout corretamente.

**Solução:** Substituir pelo `wofi --dmenu`:

```bash
cliphist list | wofi --dmenu | cliphist decode | wtype -
```

---

## NVIDIA — Hyprland não inicia ou tela preta

**Sintoma:** Hyprland falha ao iniciar ou exibe tela preta com GPU NVIDIA.

**Causa:** Módulos NVIDIA não carregados ou parâmetros de kernel ausentes.

**Solução:** Verificar checklist:

```bash
# 1. Verificar driver ativo
lsmod | grep nvidia

# 2. Verificar parâmetros de kernel
cat /etc/kernel/cmdline
# Deve conter: nvidia_drm.modeset=1 nvidia_drm.fbdev=1

# 3. Verificar módulos no mkinitcpio.conf
grep MODULES /etc/mkinitcpio.conf
# Deve conter: (nvidia nvidia_modeset nvidia_uvm nvidia_drm)
# kms NÃO deve estar nos HOOKS

# 4. Reconstruir imagem de kernel se necessário
sudo mkinitcpio -P
```

Consulte [nvidia-driver-setup.md](nvidia-driver-setup.md) para o processo completo.
