# Configuração do Gerenciador de Arquivos — Thunar

**Atalho:** `Super + E`  
**Tema:** Catppuccin Macchiato Blue (via GTK)

---

## Contexto

O **Thunar** foi escolhido por ser o gerenciador de arquivos mais leve com
interface gráfica completa — poucas dependências GTK3, sem frameworks pesados
como KDE ou GNOME, e suporte nativo a dark mode e tema via sistema GTK.

---

## Instalação

```bash
sudo pacman -S thunar
```

---

## Configuração no Hyprland

Em `~/.config/hypr/hyprland.lua`:

```lua
local fileManager = "thunar"
```

O atalho `Super + E` abre o Thunar. Ele abre como janela normal (não flutuante).

---

## Tema — GTK3 e GTK4

O tema é aplicado globalmente para todos os aplicativos GTK do sistema.

**`~/.config/gtk-3.0/settings.ini`** e **`~/.config/gtk-4.0/settings.ini`:**

```ini
[Settings]
gtk-application-prefer-dark-theme=true
gtk-theme-name=catppuccin-macchiato-blue-standard+default
gtk-icon-theme-name=Adwaita
gtk-font-name=Noto Sans 10
```

**Pacote do tema:**
```bash
yay -S catppuccin-gtk-theme-macchiato
```

> Esses arquivos afetam todos os apps GTK3 e GTK4 do sistema — Thunar,
> blueman, pavucontrol e nm-connection-editor.
