# Configuração do SwayNotificationCenter (swaync)

**Versão:** swaync 0.12.6  
**Atalho do painel:** `Super + N`  
**Tema:** Catppuccin Macchiato

---

## Contexto

O **swaync** é o daemon de notificações mais popular no ecossistema Hyprland.
Além de exibir popups de notificação, oferece um painel lateral com histórico
de notificações e suporte a Do Not Disturb.

---

## Instalação

```bash
sudo pacman -S swaync
```

---

## Autostart no Hyprland

Adicionado em `~/.config/hypr/hyprland.lua`:

```lua
hl.on("hyprland.start", function ()
    hl.exec_cmd("swaync")
end)
```

---

## Atalho de teclado

```lua
hl.bind(mainMod .. " + N", hl.dsp.exec_cmd("swaync-client -t"))
```

`swaync-client -t` alterna (toggle) o painel lateral de notificações.

### Outros comandos úteis do swaync-client

| Comando | Ação |
|---|---|
| `swaync-client -t` | Abre/fecha o painel |
| `swaync-client -d` | Ativa/desativa Do Not Disturb |
| `swaync-client -C` | Limpa todas as notificações |
| `swaync-client -c` | Conta notificações não lidas |

---

## Tema — Catppuccin Macchiato

CSS customizado em `~/.config/swaync/style.css` com as cores Catppuccin Macchiato.

| Variável | Cor | Uso |
|---|---|---|
| `--cc-bg` | `rgba(30, 32, 48, 0.92)` | Fundo do painel |
| `--noti-bg` | `rgb(54, 58, 79)` | Fundo das notificações |
| `--noti-bg-hover` | `rgb(73, 77, 100)` | Hover |
| `--text-color` | `rgb(202, 211, 245)` | Texto |
| `--bg-selected` | `rgb(138, 173, 244)` | Item selecionado (Blue) |
| Título do painel | `rgb(198, 160, 246)` | Mauve |

Para recarregar o CSS após alterações:

```bash
swaync-client -R && swaync-client -rs
```

---

## Arquivos de configuração

| Arquivo | Caminho |
|---|---|
| Config padrão | `/etc/xdg/swaync/config.json` |
| CSS padrão | `/etc/xdg/swaync/style.css` |
| CSS customizado | `~/.config/swaync/style.css` |
