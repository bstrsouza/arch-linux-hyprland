# Configuração do Hyprlock

**Arquivo:** `~/.config/hypr/hyprlock.conf`  
**Atalho:** `Super + L`  
**Estilo:** Minimalista, Catppuccin Macchiato

---

## Contexto

O **hyprlock** é o screen locker oficial do Hyprland. É acionado de duas formas:

- **Teclado:** `Super + L`
- **Menu de energia:** botão Lock no wlogout

---

## Instalação

```bash
sudo pacman -S hyprlock
```

---

## Atalho no Hyprland

Adicionado em `~/.config/hypr/hyprland.lua`:

```lua
hl.bind(mainMod .. " + L", hl.dsp.exec_cmd("hyprlock"))
```

---

## Configuração — ~/.config/hypr/hyprlock.conf

```ini
general {
    disable_loading_bar = true
    hide_cursor = true
    grace = 0
}

background {
    monitor =
    color = rgba(24, 25, 38, 1.0)
}

# Hora
label {
    monitor =
    text = cmd[update:1000] echo "$(date +%H:%M)"
    color = rgba(202, 211, 245, 1.0)
    font_size = 96
    font_family = JetBrainsMono Nerd Font Bold
    position = 0, 160
    halign = center
    valign = center
}

# Data
label {
    monitor =
    text = cmd[update:60000] echo "$(LC_TIME=pt_BR.UTF-8 date '+%A, %d de %B de %Y')"
    color = rgba(183, 189, 248, 0.8)
    font_size = 20
    font_family = JetBrainsMono Nerd Font
    position = 0, 60
    halign = center
    valign = center
}

# Campo de senha
input-field {
    monitor =
    size = 320, 52
    outline_thickness = 2
    dots_size = 0.25
    dots_spacing = 0.2
    outer_color = rgba(73, 77, 100, 1.0)
    inner_color = rgba(36, 39, 58, 1.0)
    font_color = rgba(202, 211, 245, 1.0)
    fade_on_empty = true
    placeholder_text = <i>Senha</i>
    rounding = 10
    check_color = rgba(166, 218, 149, 1.0)
    fail_color = rgba(237, 135, 150, 1.0)
    fail_text = <i>Senha incorreta</i>
    capslock_color = rgba(245, 169, 127, 1.0)
    position = 0, -60
    halign = center
    valign = center
}
```

---

## Elementos da tela

| Elemento | Descrição |
|---|---|
| Fundo | Cor sólida `#24273a` (Catppuccin base) |
| Hora | Fonte 96pt, atualizada a cada segundo |
| Data | Formato pt-BR (`segunda-feira, 22 de junho de 2025`), atualizada a cada minuto |
| Campo de senha | Bordas arredondadas, feedback visual de erro (vermelho) e Caps Lock (laranja) |

---

## Cores utilizadas (Catppuccin Macchiato)

| Elemento | Cor |
|---|---|
| Fundo | `#24273a` (base) |
| Hora | `#cad3f5` (text) |
| Data | `#b7bdf8` (lavender) |
| Borda do input | `#494d64` (surface1) |
| Interior do input | `#24273a` (base) |
| Senha correta | `#a6da95` (green) |
| Senha incorreta | `#ed8796` (red) |
| Caps Lock | `#f5a97f` (peach) |
