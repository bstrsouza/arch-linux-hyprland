# Instalação e Configuração do Hyprland

**Versão:** 0.55.4  
**Config API:** Lua (mudança introduzida no 0.47+)  
**Hardware:** Intel Iris Xe (display) + RTX 4070 Mobile (PRIME offloading)

---

## Contexto

O **Hyprland** é um compositor Wayland dinâmico com animações fluidas e alta
configurabilidade. A partir da versão 0.47, a configuração migrou do formato
`.conf` para **Lua**, o que traz mais flexibilidade mas requer atenção às
mudanças de API entre versões.

> **Nota:** Este documento pressupõe que o Arch Linux já está instalado com
> sistema de arquivos BTRFS e o driver NVIDIA configurado.
> Consulte [nvidia-driver-setup.md](nvidia-driver-setup.md) antes de prosseguir.

---

## Instalação

```bash
sudo pacman -S hyprland xdg-desktop-portal-hyprland
```

- **hyprland** — o compositor
- **xdg-desktop-portal-hyprland** — necessário para compartilhamento de tela,
  captura de janelas e integração com apps Wayland

---

## Iniciando pela primeira vez

Sem um display manager (SDDM), o Hyprland é iniciado direto do TTY:

```bash
Hyprland
```

Para inicialização automática com SDDM, consulte [sddm-setup.md](sddm-setup.md).

---

## Arquivo de configuração

**Localização:** `~/.config/hypr/hyprland.lua`

O Hyprland cria um arquivo de exemplo na primeira execução. A estrutura básica:

```
~/.config/hypr/
├── hyprland.lua      — configuração principal
├── hyprlock.conf     — configuração do bloqueador de tela
└── hypridle.conf     — configuração do daemon de inatividade
```

---

## Configurações aplicadas

### Monitor

```lua
hl.monitor({
    output   = "",
    mode     = "2880x1800",
    position = "auto",
    scale    = "1.5",
})
```

Escala 1.5x para o display 2880x1800 do Samsung 960XFH. `output = ""` é uma regra
coringa — se aplica a qualquer monitor sem uma regra própria, útil como fallback.

> Para configurar múltiplos monitores (posição, resolução por tela, mover
> workspaces entre monitores), consulte [monitors-setup.md](monitors-setup.md).

---

### Variáveis de ambiente — NVIDIA

```lua
hl.env("LIBVA_DRIVER_NAME", "nvidia")
hl.env("GBM_BACKEND", "nvidia-drm")
hl.env("__GLX_VENDOR_LIBRARY_NAME", "nvidia")
hl.env("NVD_BACKEND", "direct")
```

Necessário para que o Hyprland use corretamente a GPU NVIDIA via PRIME offloading.

---

### Programas padrão

```lua
local terminal    = "kitty"
local fileManager = "thunar"
local menu        = "wofi --show drun"
local browser     = "firefox"
```

---

### Autostart

```lua
hl.on("hyprland.start", function ()
    hl.exec_cmd("awww-daemon")
    hl.exec_cmd("awww img /usr/share/hypr/wall0.png")
    hl.exec_cmd("waybar > /dev/null 2>&1")
    hl.exec_cmd("swaync")
    hl.exec_cmd("hypridle")
    hl.exec_cmd("wl-paste --type text --watch cliphist store")
    hl.exec_cmd("wl-paste --type image --watch cliphist store")
    hl.exec_cmd("/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1")
end)
```

---

### Aparência

```lua
general = {
    gaps_in     = 5,
    gaps_out    = 10,
    border_size = 2,
    col = {
        active_border   = { colors = {"rgba(c6a0f6ee)", "rgba(8aadf4ee)"}, angle = 45 },
        inactive_border = "rgba(363a4faa)",
    },
    layout = "dwindle",
},
decoration = {
    rounding = 10,
},
```

Bordas com gradiente Mauve → Blue (Catppuccin Macchiato). Cantos arredondados de 10px.

---

### Teclado e touchpad

```lua
input = {
    kb_layout  = "br",
    kb_variant = "abnt2",
    touchpad = {
        natural_scroll = false,
    },
},
```

---

### Window rules — janelas flutuantes

```lua
hl.window_rule({ name = "blueman-float",
    match = { class = "blueman-manager" },
    float = true, size = "700 500", center = true })

hl.window_rule({ name = "pavucontrol-float",
    match = { class = "org.pulseaudio.pavucontrol" },
    float = true, size = "700 500", center = true })

hl.window_rule({ name = "nm-connection-editor-float",
    match = { class = "nm-connection-editor" },
    float = true, size = "700 500", center = true })
```

---

### Keybindings principais

```lua
local mainMod = "SUPER"

hl.bind(mainMod .. " + T", hl.dsp.exec_cmd(terminal))
hl.bind(mainMod .. " + R", hl.dsp.exec_cmd(menu))
hl.bind(mainMod .. " + E", hl.dsp.exec_cmd(fileManager))
hl.bind(mainMod .. " + Q", hl.dsp.window.close())
hl.bind(mainMod .. " + L", hl.dsp.exec_cmd("hyprlock"))
hl.bind(mainMod .. " + N", hl.dsp.exec_cmd("swaync-client -t"))
hl.bind(mainMod .. " + J", hl.dsp.layout("togglesplit"))
hl.bind(mainMod .. " + F",         hl.dsp.window.float({ action = "toggle" }))
hl.bind(mainMod .. " + SHIFT + F", hl.dsp.window.fullscreen())
hl.bind(mainMod .. " + V", hl.dsp.exec_cmd(
    "sh -c 'cliphist list | wofi --dmenu | cliphist decode | wtype -'"))

-- Screenshots
hl.bind("Print",               hl.dsp.exec_cmd("hyprshot -m region -o ~/Pictures/Screenshots"))
hl.bind(mainMod .. " + Print", hl.dsp.exec_cmd("hyprshot -m window -o ~/Pictures/Screenshots"))
hl.bind("SHIFT + Print",       hl.dsp.exec_cmd("hyprshot -m output -o ~/Pictures/Screenshots"))

-- Workspaces
for i = 1, 10 do
    local key = i % 10
    hl.bind(mainMod .. " + " .. key,         hl.dsp.focus({ workspace = i }))
    hl.bind(mainMod .. " + SHIFT + " .. key, hl.dsp.window.move({ workspace = i }))
end

-- Mover workspace entre monitores
hl.bind(mainMod .. " + SHIFT + right", hl.dsp.workspace.move({ monitor = "+1" }))
hl.bind(mainMod .. " + SHIFT + left",  hl.dsp.workspace.move({ monitor = "-1" }))
```

Detalhes sobre múltiplos monitores em [monitors-setup.md](monitors-setup.md).

---

## Nota importante — Hyprland 0.55 e dispatch

O Hyprland 0.55 com config Lua **mudou o protocolo de dispatch**. O formato
`hyprctl dispatch exec <app>` não funciona mais. Use chamadas diretas:

```jsonc
// Errado
"on-click": "hyprctl dispatch exec blueman-manager"

// Correto
"on-click": "blueman-manager"
```

Isso afeta configurações de Waybar e outros apps que chamam programas via Hyprland.

---

## Recarregar configuração

```bash
hyprctl reload
```
