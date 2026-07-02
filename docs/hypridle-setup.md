# Configuração do Idle Daemon — hypridle

**Versão:** 0.1.7  
**Pacote:** hypridle (repositório oficial Arch)  
**Ecossistema:** Hyprland nativo

---

## Contexto

O **hypridle** é o daemon de inatividade oficial do Hyprland. Ele monitora o tempo sem interação do usuário e executa ações progressivas: reduzir brilho, bloquear a tela, desligar o display e suspender o sistema.

Integra-se nativamente com o **hyprlock** (bloqueio de tela) e o **loginctl** (gerenciador de sessão do systemd).

---

## Instalação

```bash
sudo pacman -S hypridle
```

---

## Autostart

Adicionado ao `~/.config/hypr/hyprland.lua` para iniciar junto com o Hyprland:

```lua
hl.on("hyprland.start", function ()
    -- ...outros processos...
    hl.exec_cmd("hypridle")
end)
```

---

## Configuração

**`~/.config/hypr/hypridle.conf`:**

```ini
general {
    lock_cmd = pidof hyprlock || hyprlock
    before_sleep_cmd = loginctl lock-session
    after_sleep_cmd = hyprctl dispatch dpms on
    ignore_dbus_inhibit = false
}

# 4 min: reduz brilho para 10%
listener {
    timeout = 240
    on-timeout = brightnessctl -s set 10%
    on-resume = brightnessctl -r
}

# 5 min: bloqueia a tela
listener {
    timeout = 300
    on-timeout = loginctl lock-session
    on-resume = hyprctl dispatch dpms on
}

# 8 min: desliga o display
listener {
    timeout = 480
    on-timeout = hyprctl dispatch dpms off
    on-resume = hyprctl dispatch dpms on
}

# 15 min: suspende o sistema
listener {
    timeout = 900
    on-timeout = systemctl suspend
}
```

### Explicação das opções

**Seção `general`:**

| Opção | Descrição |
|---|---|
| `lock_cmd` | Comando executado ao bloquear. `pidof hyprlock \|\| hyprlock` evita abrir duas instâncias |
| `before_sleep_cmd` | Executado antes de suspender — garante que a tela seja bloqueada |
| `after_sleep_cmd` | Executado ao retomar — religa o display |
| `ignore_dbus_inhibit` | Se `true`, ignora apps que pedem para não bloquear (ex: vídeo em fullscreen) |

**Sequência de listeners:**

| Timeout | Ação | Retorno |
|---|---|---|
| 4 min (240s) | Brilho cai para 10% | Brilho restaurado |
| 5 min (300s) | Tela bloqueada (hyprlock) | Display religado |
| 8 min (480s) | Display desligado | Display religado |
| 15 min (900s) | Sistema suspenso | — |

> Para ajustar os tempos, edite os valores de `timeout` em `~/.config/hypr/hypridle.conf`. O hypridle relê o arquivo automaticamente.

---

## Teste sem reiniciar

Para iniciar o hypridle na sessão atual:

```bash
hypridle &
```

Para verificar se está rodando:

```bash
pidof hypridle
```
