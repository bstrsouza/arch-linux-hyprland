# Quick Start — Do zero ao ambiente funcionando

Este guia assume que você tem o **Arch Linux instalado** com sistema de arquivos
**BTRFS** e está no TTY pela primeira vez. Siga a ordem abaixo.

> Para dual boot com Windows, consulte a documentação do Arch Wiki sobre
> particionamento com Windows pré-instalado.

---

## Etapa 1 — Pré-requisitos

```bash
sudo pacman -S git base-devel
```

---

## Etapa 2 — Instalar o yay (AUR helper)

```bash
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
cd .. && rm -rf yay
```

Consulte [yay-setup.md](yay-setup.md) para detalhes.

---

## Etapa 3 — Driver NVIDIA (apenas RTX/GTX)

> Pule esta etapa se não tiver GPU NVIDIA.

```bash
sudo pacman -S nvidia-open nvidia-utils lib32-nvidia-utils
```

Edite `/etc/kernel/cmdline` adicionando ao final:
```
nvidia_drm.modeset=1 nvidia_drm.fbdev=1
```

Edite `/etc/mkinitcpio.conf`:
```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
HOOKS=(base udev autodetect microcode modconf keyboard keymap consolefont block filesystems fsck)
```

Remova `kms` dos HOOKS se estiver presente. Reconstrua:
```bash
sudo mkinitcpio -P
sudo reboot
```

Consulte [nvidia-driver-setup.md](nvidia-driver-setup.md) para detalhes.

---

## Etapa 4 — Instalar o Hyprland

```bash
sudo pacman -S hyprland xdg-desktop-portal-hyprland
```

Teste iniciando pelo TTY:
```bash
Hyprland
```

Consulte [hyprland-setup.md](hyprland-setup.md) para a configuração completa.

---

## Etapa 5 — Display Manager (SDDM)

```bash
sudo pacman -S sddm
sudo systemctl enable sddm
yay -S catppuccin-sddm-theme-macchiato
```

Configure `/etc/sddm.conf`:
```ini
[Theme]
Current=catppuccin-macchiato-blue
```

Consulte [sddm-setup.md](sddm-setup.md) para detalhes.

---

## Etapa 6 — Shell (Zsh + Starship)

```bash
sudo pacman -S zsh starship zsh-autosuggestions zsh-syntax-highlighting
chsh -s /bin/zsh
```

Configure `~/.zshrc`. Consulte [zsh-setup.md](zsh-setup.md) para detalhes.

---

## Etapa 7 — Ferramentas do desktop

```bash
sudo pacman -S waybar swaync hyprlock hypridle hyprshot
sudo pacman -S kitty wofi wl-clipboard cliphist wtype
sudo pacman -S thunar swww
sudo pacman -S blueman pavucontrol nm-connection-editor
sudo pacman -S brightnessctl playerctl
sudo pacman -S polkit-gnome
yay -S wlogout spotify
```

---

## Etapa 8 — Tema Catppuccin

```bash
yay -S catppuccin-gtk-theme-macchiato
```

Aplique em `~/.config/gtk-3.0/settings.ini` e `~/.config/gtk-4.0/settings.ini`:
```ini
[Settings]
gtk-application-prefer-dark-theme=true
gtk-theme-name=catppuccin-macchiato-blue-standard+default
gtk-icon-theme-name=Adwaita
gtk-font-name=Noto Sans 10
```

Consulte [catppuccin-setup.md](catppuccin-setup.md) para aplicar em todos os componentes.

---

## Etapa 9 — Utilitários

```bash
# Impressora
sudo pacman -S cups cups-pdf ghostscript gsfonts
sudo systemctl enable --now cups
yay -S samsung-unified-driver-printer   # se tiver impressora Samsung

# Backup
sudo pacman -S timeshift
# Abra timeshift-launcher, selecione BTRFS e configure o agendamento
```

---

## Etapa 10 — Configurar arquivos

Copie ou adapte os arquivos de configuração deste repositório para `~/.config/`:

```
~/.config/hypr/hyprland.lua
~/.config/hypr/hyprlock.conf
~/.config/hypr/hypridle.conf
~/.config/waybar/config.jsonc
~/.config/waybar/style.css
~/.config/kitty/kitty.conf
~/.config/swaync/style.css
~/.config/wofi/config
~/.config/wofi/style.css
~/.config/wlogout/layout
~/.config/wlogout/style.css
```

---

## Resultado esperado

Ao reiniciar após todas as etapas:

- Tela de login SDDM com tema Catppuccin Macchiato
- Hyprland iniciando automaticamente
- Waybar com todos os módulos funcionando
- Tema unificado em todos os apps
