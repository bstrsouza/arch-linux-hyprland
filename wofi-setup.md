# Configuração do App Launcher — wofi

**Atalho:** `Super + R` e botão na Waybar  
**Tema:** Catppuccin Macchiato

---

## Contexto

O **wofi** é um launcher Wayland nativo baseado em GTK. Foi escolhido por ser
leve e capaz de fazer as duas funções necessárias:

- **App launcher** — abre aplicativos instalados (`--show drun`)
- **Dmenu** — seletor de lista para o histórico de clipboard (`--dmenu`)

Substituiu o `hyprlauncher`, centralizando as duas funções em um único app.

---

## Instalação

```bash
sudo pacman -S wofi
sudo pacman -Rns hyprlauncher
```

---

## Integração com Hyprland

Em `~/.config/hypr/hyprland.lua`:

```lua
local menu = "wofi --show drun"
```

O atalho `Super+R` já estava configurado e passa a abrir o wofi.

---

## Integração com Waybar

Em `~/.config/waybar/config.jsonc`:

```json
"custom/menu": {
    "format": "",
    "on-click": "wofi --show drun"
}
```

---

## Integração com Clipboard

Usado como seletor no fluxo do cliphist. Consulte `clipboard-setup.md`:

```bash
cliphist list | wofi --dmenu | cliphist decode | wtype -
```

---

## Arquivos de configuração

### `~/.config/wofi/config`

```ini
width=600
height=400
location=center
prompt=Buscar...
allow_images=true
image_size=24
gtk_dark=true
insensitive=true
```

### `~/.config/wofi/style.css` — Catppuccin Macchiato

| Elemento | Cor |
|---|---|
| Fundo da janela | `rgba(24, 25, 38, 0.95)` — Crust |
| Campo de busca | `rgba(54, 58, 79, 0.8)` — Surface0 |
| Texto | `#cad3f5` — Text |
| Item selecionado | `#8aadf4` — Blue |
| Texto selecionado | `#24273a` — Base |
| Fonte | JetBrainsMono Nerd Font, 14px |
