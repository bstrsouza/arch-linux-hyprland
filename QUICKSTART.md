# Quick Start — Do zero ao ambiente funcionando

Este guia assume que você tem o **Arch Linux instalado** com sistema de arquivos **BTRFS** e está no TTY pela primeira vez. Siga a ordem abaixo.

> Para dual boot com Windows, consulte a documentação do Arch Wiki sobre particionamento com Windows pré-instalado.

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

Consulte [yay-setup.md](docs/yay-setup.md) para detalhes.

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

Consulte [nvidia-driver-setup.md](docs/nvidia-driver-setup.md) para detalhes.

---

## Etapa 4 — Instalar o Hyprland

```bash
sudo pacman -S hyprland xdg-desktop-portal-hyprland
```

Teste iniciando pelo TTY:
```bash
Hyprland
```

Consulte [hyprland-setup.md](docs/hyprland-setup.md) para a configuração completa.

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

Consulte [sddm-setup.md](docs/sddm-setup.md) para detalhes.

---

## Etapa 6 — Shell (Zsh + Starship)

```bash
sudo pacman -S zsh starship zsh-autosuggestions zsh-syntax-highlighting
chsh -s /bin/zsh
```

Configure `~/.zshrc`. Consulte [zsh-setup.md](docs/zsh-setup.md) para detalhes.

---

## Etapa 7 — Ferramentas do desktop

```bash
sudo pacman -S waybar swaync hyprlock hypridle hyprshot
sudo pacman -S kitty wofi wl-clipboard cliphist wtype
sudo pacman -S ttf-jetbrains-mono-nerd # fonte usada em kitty, hyprlock, wofi, waybar e no preset do Starship
sudo pacman -S thunar swww
sudo pacman -S blueman pavucontrol nm-connection-editor networkmanager-dmenu
sudo pacman -S brightnessctl playerctl
sudo pacman -S polkit-gnome
sudo pacman -S chromium # WhatsApp Web isolado em duas contas, ver Etapa 9
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

Consulte [catppuccin-setup.md](docs/catppuccin-setup.md) para aplicar em todos os componentes.

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

# VPN (OpenVPN)
sudo pacman -S openvpn networkmanager-openvpn
sudo systemctl restart NetworkManager
# Importe o .ovpn pelo nm-connection-editor ou networkmanager_dmenu (waybar)
```

Consulte [vpn-openvpn-setup.md](docs/vpn-openvpn-setup.md) para detalhes.

```bash
# WhatsApp (duas contas isoladas via Chromium — chromium instalado na Etapa 7)
mkdir -p ~/.config/chromium-whatsapp1 ~/.config/chromium-whatsapp2
# Super+W / Super+Shift+W (binds já vêm no hyprland.lua copiado na Etapa 10)
# No primeiro uso de cada conta, escaneie o QR code do WhatsApp Web
```

Consulte [whatsapp-chromium-setup.md](docs/whatsapp-chromium-setup.md) para detalhes.

---

## Etapa 10 — Configurar arquivos

Copie ou adapte os arquivos de configuração deste repositório para `~/.config/` (e `~/.zshrc`):

```
# Hyprland
~/.config/hypr/hyprland.lua
~/.config/hypr/hyprlock.conf
~/.config/hypr/hypridle.conf

# Waybar
~/.config/waybar/config.jsonc
~/.config/waybar/style.css
~/.config/networkmanager-dmenu/config.ini

# Kitty
~/.config/kitty/kitty.conf
~/.config/kitty/current-theme.conf

# swaync, Wofi, wlogout
~/.config/swaync/style.css
~/.config/wofi/config
~/.config/wofi/style.css
~/.config/wlogout/layout
~/.config/wlogout/style.css

# Tema GTK / Catppuccin (ver catppuccin-setup.md)
~/.config/gtk-3.0/settings.ini
~/.config/gtk-4.0/settings.ini
~/.config/dconf/user

# Shell
~/.zshrc
~/.config/starship.toml
```

> Arquivos de sistema (`/etc/sddm.conf`, `/etc/mkinitcpio.conf`, `/etc/kernel/cmdline` etc.) já foram tratados nas etapas 3 e 5. O webcam Intel IPU6/OV02C10 só se aplica a esse hardware específico — veja [webcam-setup.md](docs/webcam-setup.md). O brilho da tela tem etapa própria a seguir.

---

## Etapa 11 — Brilho da tela (hardware específico — Samsung 960XFH)

> Necessário apenas neste modelo de notebook (painel eDP com controle de brilho via DPCD/AUX). Se `Fn+F2`/`Fn+F3` já mudarem o brilho fisicamente depois da Etapa 10, pule esta etapa.

Se o percentual mudar no `brightnessctl -l` e no indicador da Waybar, mas a tela não responder fisicamente, adicione o parâmetro de kernel que força a interface DPCD proprietária da Intel:

```bash
sudo sed -i 's/$/ i915.enable_dpcd_backlight=3/' /etc/kernel/cmdline
sudo mkinitcpio -P
sudo reboot
```

Consulte [brightness-setup.md](docs/brightness-setup.md) para o diagnóstico completo (como descartar GPU híbrida/driver Samsung como causa) e os valores intermediários testados antes de chegar em `3`.

---

## Resultado esperado

Ao reiniciar após todas as etapas:

- Tela de login SDDM com tema Catppuccin Macchiato
- Hyprland iniciando automaticamente
- Waybar com todos os módulos funcionando
- Tema unificado em todos os apps
