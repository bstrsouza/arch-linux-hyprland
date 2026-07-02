# Configuração do Terminal — Kitty

**Versão:** 0.47.1  
**Atalho:** `Super + T`  
**Tema:** Catppuccin-Macchiato  
**Fonte:** JetBrainsMono Nerd Font

---

## Contexto

O **kitty** é um emulador de terminal GPU-accelerated, rápido e altamente customizável. Já veio pré-configurado com o tema Catppuccin-Macchiato e a fonte JetBrainsMono Nerd Font, consistentes com o restante do ambiente.

---

## Instalação

```bash
sudo pacman -S kitty
```

---

## Configuração no Hyprland

Em `~/.config/hypr/hyprland.lua`:

```lua
local terminal = "kitty"
```

O atalho `Super + T` abre o terminal.

---

## Arquivos de configuração

```
~/.config/kitty/kitty.conf          — configuração principal
~/.config/kitty/current-theme.conf  — tema ativo (Catppuccin-Macchiato)
```

### Configurações ativas em kitty.conf

```
include current-theme.conf
font_family      family="JetBrainsMono Nerd Font"
bold_font        auto
italic_font      auto
bold_italic_font auto
```

### Tema — Catppuccin-Macchiato

Aplicado via `current-theme.conf`. Cores principais:

| Elemento | Cor |
|---|---|
| Foreground | `#CAD3F5` |
| Background | `#24273A` |
| Cursor | `#F4DBD6` |

Para trocar o tema pelo terminal:

```bash
kitty +kitten themes
```
