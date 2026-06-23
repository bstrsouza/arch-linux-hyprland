# Tema Catppuccin Macchiato

**Palette:** Macchiato  
**Accent:** Blue (`#8aadf4`)  
**Escopo:** Todo o ambiente — Hyprland, GTK, SDDM, swaync, wlogout, Kitty, Waybar, Hyprlock

---

## Contexto

O **Catppuccin** é uma palette de cores pastel escura amplamente adotada na comunidade Linux.
A variante **Macchiato** oferece um fundo médio-escuro (`#24273a`) com cores suaves e bem
contrastadas, sem ser tão escura quanto o Mocha.

A unificação do tema em todos os componentes garante consistência visual entre a tela de login,
o compositor, os apps GTK, as notificações e o power menu.

---

## Cores principais (Macchiato)

| Nome      | Hex       | Uso                          |
|-----------|-----------|------------------------------|
| Base      | `#24273a` | Fundo principal              |
| Mantle    | `#1e2030` | Fundo secundário             |
| Crust     | `#181926` | Fundo mais escuro            |
| Surface0  | `#363a4f` | Bordas inativas, cards       |
| Surface1  | `#494d64` | Hover, elementos secundários |
| Text      | `#cad3f5` | Texto principal              |
| Blue      | `#8aadf4` | Accent principal             |
| Mauve     | `#c6a0f6` | Accent secundário (bordas)   |
| Lavender  | `#b7bdf8` | Destaque suave               |

---

## Componentes configurados

### 1. Hyprland — bordas de janela

Em `~/.config/hypr/hyprland.lua`:

```lua
col = {
    active_border   = { colors = {"rgba(c6a0f6ee)", "rgba(8aadf4ee)"}, angle = 45 },
    inactive_border = "rgba(363a4faa)",
},
```

Gradiente de 45° entre Mauve e Blue na borda ativa. Borda inativa usa Surface0.

---

### 2. GTK 3 e GTK 4

**Pacote instalado:**

```bash
yay -S catppuccin-gtk-theme-macchiato
```

Instala os temas em `/usr/share/themes/catppuccin-macchiato-*/`.

**`~/.config/gtk-3.0/settings.ini`** e **`~/.config/gtk-4.0/settings.ini`:**

```ini
[Settings]
gtk-application-prefer-dark-theme=true
gtk-theme-name=catppuccin-macchiato-blue-standard+default
gtk-icon-theme-name=Adwaita
gtk-font-name=Noto Sans 10
```

Afeta todos os apps GTK: Thunar, blueman, pavucontrol, nm-connection-editor.

---

### 3. SDDM — tela de login

**Pacote instalado:**

```bash
yay -S catppuccin-sddm-theme-macchiato
```

Instala os temas em `/usr/share/sddm/themes/catppuccin-macchiato-*/`.

**`/etc/sddm.conf`:**

```ini
[Theme]
Current=catppuccin-macchiato-blue
```

O tema é aplicado na próxima reinicialização do sistema.

---

### 4. swaync — notificações

CSS em `~/.config/swaync/style.css` com variáveis Catppuccin Macchiato:

| Variável         | Cor                        |
|------------------|----------------------------|
| `--cc-bg`        | `rgba(30, 32, 48, 0.92)`   |
| `--noti-bg`      | `54, 58, 79` (Surface0)    |
| `--noti-bg-hover`| `rgb(73, 77, 100)`          |
| `--text-color`   | `rgb(202, 211, 245)`        |
| `--bg-selected`  | `rgb(138, 173, 244)` (Blue) |

Para recarregar após alterar o CSS:

```bash
swaync-client -R && swaync-client -rs
```

---

### 5. wlogout — power menu

CSS em `~/.config/wlogout/style.css`:

- Fundo da janela: `rgba(24, 25, 38, 0.9)` (Crust semitransparente)
- Botões: Surface0 com borda suave
- Hover: Surface1 com borda Mauve
- Ícones padrão do wlogout em `/usr/share/wlogout/icons/`

---

### 6. Kitty — terminal

Já configurado com Catppuccin Macchiato via `~/.config/kitty/current-theme.conf`.
Consulte `kitty-setup.md` para detalhes.

---

### 7. Waybar e Hyprlock

Já configurados com cores Macchiato diretamente nos arquivos CSS/conf.
Consulte `waybar-setup.md` e `hyprlock-setup.md` para detalhes.

---

### 8. wofi — launcher e clipboard picker

CSS em `~/.config/wofi/style.css` com cores Catppuccin Macchiato.
Consulte `wofi-setup.md` para detalhes.

---

## Resumo de arquivos modificados

| Arquivo | Alteração |
|---|---|
| `~/.config/hypr/hyprland.lua` | Bordas Mauve + Blue |
| `~/.config/gtk-3.0/settings.ini` | Tema GTK Macchiato Blue |
| `~/.config/gtk-4.0/settings.ini` | Tema GTK Macchiato Blue |
| `/etc/sddm.conf` | Tema SDDM Macchiato Blue |
| `~/.config/swaync/style.css` | CSS Catppuccin Macchiato |
| `~/.config/wlogout/style.css` | CSS Catppuccin Macchiato |
| `~/.config/wofi/style.css` | CSS Catppuccin Macchiato |
