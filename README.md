# Arch Linux + Hyprland — Ambiente de Trabalho Completo

Um ambiente de trabalho moderno, leve e visualmente unificado construído sobre **Arch Linux** e **Hyprland**, com tema **Catppuccin Macchiato** aplicado em todos os componentes.

---

## Sobre o projeto

Este repositório documenta a configuração completa de um ambiente Linux profissional, incluindo cada decisão tomada, problema encontrado e solução aplicada.

**Objetivo:** Servir como referência para replicar ou adaptar este ambiente em qualquer máquina com Arch Linux, economizando horas de pesquisa e tentativa e erro.

**Público-alvo:** Usuários iniciantes a intermediários do Linux que queiram montar um ambiente Hyprland funcional, bonito e bem documentado.

---

## O que você vai ter ao final

- Tela de login SDDM com tema Catppuccin Macchiato
- Compositor Hyprland 0.55 com animações fluidas
- Barra de status Waybar totalmente configurada
- Terminal Kitty, gerenciador de arquivos Thunar, launcher Wofi
- Notificações (swaync), bloqueio de tela (hyprlock), idle automático (hypridle)
- Screenshot, clipboard com histórico, menu de energia
- Tema Catppuccin Macchiato unificado em todos os apps
- Driver NVIDIA com suporte a PRIME offloading
- Backup automático com Timeshift (BTRFS)

---

## Hardware de referência

| Componente | Especificação |
|---|---|
| Modelo | Samsung 960XFH |
| CPU | Intel Core i9-13900H (14 cores) |
| GPU | NVIDIA RTX 4070 Mobile + Intel Iris Xe |
| RAM | 32 GB |
| Armazenamento | 1 TB NVMe (BTRFS) |
| Display | 2880x1800, escala 1.5x |
| Dual boot | Windows |

---

## Por onde começar

| Documento | Quando usar |
|---|---|
| [QUICKSTART.md](QUICKSTART.md) | Instalação do zero — sequência completa passo a passo |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | Algo não funcionou — problemas conhecidos e soluções |
| [README.md](README.md) | Visão geral, índice completo e referência rápida |

---

## Índice

### Pré-requisitos

| Documento | Descrição |
|---|---|
| [Git](docs/git-setup.md) | Controle de versão e dependência para o AUR |
| [yay](docs/yay-setup.md) | AUR helper — instalar pacotes comunitários |

### Sistema base

| Documento | Descrição |
|---|---|
| [Hyprland](docs/hyprland-setup.md) | Instalação, configuração completa e notas sobre v0.55 |
| [Driver NVIDIA](docs/nvidia-driver-setup.md) | nvidia-open para RTX 4070, PRIME offloading, parâmetros de kernel |
| [SDDM](docs/sddm-setup.md) | Display manager com tema Catppuccin Macchiato Blue |
| [Tema Catppuccin](docs/catppuccin-setup.md) | Unificação visual em todos os componentes |

### Desktop e compositor

| Documento | Descrição |
|---|---|
| [Waybar](docs/waybar-setup.md) | Barra de status com todos os módulos configurados |
| [Monitores](docs/monitors-setup.md) | Múltiplos monitores — posição, escala e mover workspaces entre telas |
| [Wallpaper](docs/wallpaper-setup.md) | awww (swww) com autostart completo do Hyprland |
| [Hyprlock](docs/hyprlock-setup.md) | Bloqueio de tela minimalista, atalho `Super + L` |
| [hypridle](docs/hypridle-setup.md) | Daemon de inatividade — brilho, lock, display, suspend |
| [swaync](docs/swaync-setup.md) | Notificações com painel lateral, atalho `Super + N` |

### Aplicativos

| Documento | Descrição |
|---|---|
| [Kitty](docs/kitty-setup.md) | Terminal GPU-accelerated, atalho `Super + T` |
| [Wofi](docs/wofi-setup.md) | App launcher e seletor de clipboard, atalho `Super + R` |
| [Thunar](docs/thunar-setup.md) | Gerenciador de arquivos, atalho `Super + E` |
| [Zsh + Starship](docs/zsh-setup.md) | Shell com autocompletar, syntax highlighting e prompt |

### Utilitários

| Documento | Descrição |
|---|---|
| [Screenshot](docs/screenshot-setup.md) | hyprshot — região `Print`, janela `Super+Print`, monitor `Shift+Print` |
| [Clipboard](docs/clipboard-setup.md) | cliphist + wofi + wtype — histórico com `Super + Shift + V` |
| [Polkit Agent](docs/polkit-setup.md) | polkit-gnome — janela de autenticação para apps gráficos |
| [Impressora](docs/printer-setup.md) | CUPS + driver Samsung — USB e rede corporativa |
| [Timeshift](docs/timeshift-setup.md) | Snapshots BTRFS automáticos — diário, semanal e mensal |

---

## Atalhos de teclado

| Tecla | Ação |
|---|---|
| `Super + T` | Abrir terminal (kitty) |
| `Super + R` | Abrir launcher (wofi) |
| `Super + E` | Abrir gerenciador de arquivos (Thunar) |
| `Super + Q` | Fechar janela |
| `Super + L` | Bloquear tela (hyprlock) |
| `Super + N` | Painel de notificações (swaync) |
| `Super + F` | Alternar janela flutuante |
| `Super + Shift + F` | Fullscreen |
| `Super + J` | Alternar split do layout (dwindle) |
| `Super + V` | Histórico de clipboard |
| `Super + 1..9` | Trocar workspace |
| `Super + Shift + 1..9` | Mover janela para workspace |
| `Super + Shift + →` | Mover workspace atual para o próximo monitor |
| `Super + Shift + ←` | Mover workspace atual para o monitor anterior |
| `Super + S` | Scratchpad (workspace especial) |
| `Print` | Screenshot de região |
| `Super + Print` | Screenshot de janela |
| `Shift + Print` | Screenshot de monitor |

---

## Autostart (Hyprland)

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

## Pacotes instalados

### Pré-requisitos

```bash
sudo pacman -S git base-devel
git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si
sudo pacman -S hyprland xdg-desktop-portal-hyprland
```

### Repositório oficial (pacman)

```bash
sudo pacman -S nvidia-open nvidia-utils lib32-nvidia-utils
sudo pacman -S sddm
sudo pacman -S hyprlock hypridle hyprshot
sudo pacman -S waybar
sudo pacman -S swaync
sudo pacman -S kitty
sudo pacman -S wofi wl-clipboard cliphist wtype
sudo pacman -S thunar
sudo pacman -S zsh starship zsh-autosuggestions zsh-syntax-highlighting
sudo pacman -S blueman pavucontrol nm-connection-editor
sudo pacman -S swww
sudo pacman -S polkit-gnome
sudo pacman -S cups cups-pdf ghostscript gsfonts
sudo pacman -S timeshift
sudo pacman -S brightnessctl playerctl
sudo pacman -S usbutils
```

### AUR (yay)

```bash
yay -S catppuccin-gtk-theme-macchiato
yay -S catppuccin-sddm-theme-macchiato
yay -S spotify
yay -S wlogout
yay -S samsung-unified-driver-printer
```
