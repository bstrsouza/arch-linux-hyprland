# Configuração da Waybar

**Arquivos:** `~/.config/waybar/config.jsonc` e `~/.config/waybar/style.css`  
**Tema:** Catppuccin Macchiato  
**Autostart:** via `~/.config/hypr/hyprland.lua`

---

## Autostart no Hyprland

A Waybar é iniciada automaticamente com o Hyprland. A saída é redirecionada para `/dev/null` para evitar logs no terminal:

```lua
hl.on("hyprland.start", function ()
    hl.exec_cmd("waybar > /dev/null 2>&1")
end)
```

---

## Estrutura dos módulos

```
[  workspaces  |  janela ativa  ]  [  relógio  ]  [  spotify | bt | vol | wifi | brilho | bat |  ]
       esquerda                        centro                      direita
```

---

## Correção importante — Hyprland 0.55 + Waybar

O Hyprland 0.55 com config Lua **mudou o protocolo de dispatch**. O formato antigo `hyprctl dispatch exec <app>` não funciona mais e retorna erro de sintaxe Lua.

**Solução:** chamar os aplicativos diretamente, sem o wrapper `hyprctl dispatch exec`.

```jsonc
// Errado (Hyprland 0.55 retorna erro)
"on-click": "hyprctl dispatch exec blueman-manager"

// Correto
"on-click": "blueman-manager"
```

Isso foi corrigido em todos os módulos: `custom/menu`, `bluetooth`, `pulseaudio`, `network` e `custom/power`.

---

## Módulos

### custom/menu
Botão com o ícone do Arch que abre o `wofi` como app launcher.

```jsonc
"custom/menu": {
    "format": "",
    "on-click": "wofi --show drun"
}
```

---

### hyprland/workspaces
Exibe os workspaces ativos com ícones diferenciados por estado.

```jsonc
"hyprland/workspaces": {
    "on-click": "activate",
    "format": "{icon}",
    "format-icons": {
        "0": "",
        "active": "",
        "default": "",
        "urgent": "󰀦"
    }
}
```

> **Problema conhecido:** o clique para trocar de workspace via mouse não funciona no Waybar 0.15.0 com Hyprland 0.55. O módulo interno usa `dispatch workspace X` (formato antigo) para comunicação via socket, que foi descontinuado no Hyprland 0.55. A troca por teclado (`Super + 1..9`) funciona normalmente. Resolver após atualização do Waybar.

---

### hyprland/window
Exibe o título da janela ativa em itálico.

```jsonc
"hyprland/window": {
    "format": " <span style=\"italic\">{title}</span> ",
    "max-length": 40
}
```

---

### clock
Relógio centralizado com data e hora. Tooltip exibe calendário ao passar o mouse.

```jsonc
"clock": {
    "format": " {:%d/%m/%Y %H:%M}",
    "tooltip-format": "<tt><small>{calendar}</small></tt>"
}
```

---

### custom/spotify
Exibe artista e título da música tocando no Spotify. Some automaticamente quando o Spotify está fechado (`2>/dev/null` suprime o erro do playerctl).

```jsonc
"custom/spotify": {
    "exec": "playerctl -p spotify metadata --format '{{artist}} - {{title}}' 2>/dev/null",
    "interval": 5,
    "format": "  {}",
    "max-length": 30,
    "on-click": "playerctl -p spotify play-pause",
    "escape": true
}
```

**Dependência:** `playerctl`, `spotify` (AUR)

---

### bluetooth
Exibe ícone de bluetooth com estado (conectado / desconectado / desligado). Abre o `blueman-manager` ao clicar.

```jsonc
"bluetooth": {
    "format": "󰂯",
    "format-disabled": "󰂲",
    "format-connected": "󰂱",
    "tooltip-format": "{controller_alias}\t{controller_address}",
    "tooltip-format-connected": "{controller_alias}\t{controller_address}\n\n{device_enumerate}",
    "tooltip-format-enumerate-connected": "{device_alias}\t{device_address}",
    "on-click": "blueman-manager"
}
```

**Dependência:** `blueman`

**Window rule** (janela flutuante centralizada):
```lua
hl.window_rule({
    name   = "blueman-float",
    match  = { class = "blueman-manager" },
    float  = true,
    size   = "700 500",
    center = true,
})
```

---

### pulseaudio
Exibe ícone e percentual de volume. Abre o `pavucontrol` ao clicar.

```jsonc
"pulseaudio": {
    "format": "{icon} {volume}%",
    "format-muted": "󰝟 Mute",
    "format-icons": { "default": ["", "", ""] },
    "on-click": "pavucontrol"
}
```

**Dependência:** `pavucontrol`

**Window rule** (janela flutuante centralizada):
```lua
hl.window_rule({
    name   = "pavucontrol-float",
    match  = { class = "org.pulseaudio.pavucontrol" },
    float  = true,
    size   = "700 500",
    center = true,
})
```

---

### network
Exibe ícone WiFi e intensidade do sinal. Tooltip mostra nome da rede e gateway.

Clique esquerdo abre o `networkmanager_dmenu` (lista redes WiFi e conexões VPN/Wireguard já configuradas, com opção de **conectar/desconectar**). Clique direito abre o `nm-connection-editor`, usado apenas para criar/editar/excluir perfis de conexão — ele **não** tem opção de conectar/desconectar, por isso não serve sozinho para esse fim.

```jsonc
"network": {
    "format-wifi": "  {signalStrength}%",
    "format-ethernet": "󰈀",
    "format-linked": "󰈀 (No IP)",
    "format-disconnected": "󰖪",
    "tooltip-format": "{essid} - {ifname} via {gwaddr}",
    "on-click": "networkmanager_dmenu",
    "on-click-right": "nm-connection-editor"
}
```

**Dependências:** `nm-connection-editor`, `networkmanager-dmenu`

**Config do networkmanager-dmenu** (`~/.config/networkmanager-dmenu/config.ini`), para usar o `wofi` no lugar do `dmenu` padrão:

```ini
[dmenu]
dmenu_command = wofi --dmenu -i
highlight = True
```

**Window rule** (janela flutuante centralizada):
```lua
hl.window_rule({
    name   = "nm-connection-editor-float",
    match  = { class = "nm-connection-editor" },
    float  = true,
    size   = "700 500",
    center = true,
})
```

---

### backlight
Exibe ícone e percentual de brilho da tela. Scroll do mouse sobre o ícone ajusta o brilho (suporte nativo do módulo).

```jsonc
"backlight": {
    "device": "intel_backlight",
    "format": "{icon} {percent}%",
    "format-icons": ["󰃞", "󰃟", "󰃠"],
    "tooltip-format": "Brilho: {percent}%"
}
```

**Dependência:** `brightnessctl` (só para os atalhos de teclado — o módulo em si lê o sysfs direto)

> Ver [brightness-setup.md](brightness-setup.md) para o fix de kernel necessário nesse hardware (`i915.enable_dpcd_backlight`) — sem ele, o valor muda no sysfs/Waybar mas não tem efeito físico na tela.

---

### battery
Exibe ícone dinâmico e percentual de carga. Alerta visual em vermelho com animação piscante abaixo de 15%.

```jsonc
"battery": {
    "states": { "warning": 30, "critical": 15 },
    "format": "{icon} {capacity}%",
    "format-charging": "󱐋 {capacity}%",
    "format-icons": ["", "", "", "", ""]
}
```

---

### custom/power
Botão de energia que abre o `wlogout` em modo compacto horizontal com 4 opções: lock, suspend, reboot e shutdown.

```jsonc
"custom/power": {
    "format": "",
    "tooltip": "Power Options",
    "on-click": "wlogout -b 4 -m 500"
}
```

**Dependências:** `wlogout` (AUR), `hyprlock`

**Layout do wlogout** (`~/.config/wlogout/layout`):
```json
{ "label": "lock",     "action": "hyprlock",            "text": "Lock",     "keybind": "l" }
{ "label": "suspend",  "action": "systemctl suspend",   "text": "Suspend",  "keybind": "u" }
{ "label": "reboot",   "action": "systemctl reboot",    "text": "Reboot",   "keybind": "r" }
{ "label": "shutdown", "action": "systemctl poweroff",  "text": "Shutdown", "keybind": "s" }
```

> **Flags do wlogout:**
> - `-b 4` — exibe os 4 botões em uma única linha horizontal
> - `-m 500` — 500px de margem em volta, centralizando o conjunto na tela

---

## Pacotes instalados nesta configuração

```bash
sudo pacman -S blueman pavucontrol nm-connection-editor hyprlock wofi networkmanager-dmenu brightnessctl
yay -S spotify wlogout
```
