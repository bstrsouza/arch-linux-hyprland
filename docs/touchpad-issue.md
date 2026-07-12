# Touchpad intermitente — Intel I2C HID (IMG4100)

**Hardware:** Samsung 960XFH (Galaxy Book3 Ultra) — Intel Core i9-13900H (Raptor Lake)
**Touchpad:** ACPI HID `IMG4100:00` (vendor `4d49`, product `4150`) via `i2c_hid_acpi`, no controlador `i2c_designware.1` (`i2c-1`)
**Sistema:** Arch Linux, reproduzido nos kernels `7.1.2-arch3-1` e `7.1.3-arch1-2`

---

## Sintoma

O touchpad para de responder — sem gerar nenhum evento de cursor, nem na tela de bloqueio. Não é um problema de configuração: nenhuma regra do Hyprland, keybind, regra udev ou serviço systemd desabilita o dispositivo. Ocorre de forma intermitente entre boots — às vezes o touchpad funciona normalmente, às vezes falha já na inicialização.

---

## Diagnóstico

```bash
journalctl -k -b 0 | grep -viE "i915|drm:" | grep -iE "i2c|hid|IMG4100"
```

Nos boots em que o touchpad falha, aparece:

```
i2c_designware i2c_designware.1: controller timed out
i2c_designware i2c_designware.1: controller timed out
```

E o dispositivo nunca aparece em `/proc/bus/input/devices`. Confirmando que o driver não vinculou:

```bash
ls -la /sys/bus/i2c/drivers/i2c_hid_acpi/
# Só os arquivos padrão (bind/unbind/module/uevent) — nenhum symlink para IMG4100:00
```

Tentativa de bind manual reproduz o mesmo erro (o driver chega a tentar o `probe()`, mas a transação I2C falha):

```bash
echo "i2c-IMG4100:00" | sudo tee /sys/bus/i2c/drivers/i2c_hid_acpi/bind
# tee: .../bind: No such device or address   (ENXIO — retornado pelo próprio probe(), não pelo driver core)
```

Em boots anteriores (antes desta investigação), o mesmo sintoma já havia se manifestado com um erro diferente — `i2c_hid_get_input: incomplete report` (com tamanhos inconsistentes entre boots: 259, depois 65535) — indicando que a falha na negociação do HID descriptor pode se manifestar tanto como corrupção de dado quanto como timeout total do controlador, dependendo do boot.

**Causa provável:** race condition de timing entre o kernel e o firmware/hardware do controlador `i2c_designware.1` durante a enumeração do touchpad no boot — não uma regressão de driver/config. Confirmado por:

- Hardware fisicamente funcional (IRQ do dispositivo dispara ao toque, contador de interrupções sobe, mesmo quando o driver não está vinculado).
- Reproduzido de forma idêntica em duas versões de kernel (`7.1.2-arch3-1` e `7.1.3-arch1-2`), descartando regressão da atualização de pacotes de 2026-07-11.
- Não recuperável via rebind manual do driver, `suspend`/`resume` (s2idle) ou reboot completo — nenhum desses reinicializa a negociação do lado do firmware do jeito certo.
- Nenhuma configuração de software (Hyprland, udev, systemd) interfere — o dispositivo simplesmente não enumera no barramento I2C.

---

## Tentativas realizadas

| Tentativa | Resultado |
|---|---|
| Rebind manual do driver (`unbind`/`bind` via sysfs) | Às vezes recria o `input` device sem erro de log, mas sem gerar movimento de cursor (dado ainda corrompido internamente) |
| `systemctl suspend` + resume | Sem efeito — controlador continua retornando `controller timed out` no bind seguinte |
| Reboot completo | Sem efeito — sintoma reaparece em boots subsequentes, com formas de erro diferentes |
| Downgrade do kernel (`7.1.3-arch1-2` → `7.1.2-arch3-1`, via cache do pacman) | Sem efeito — mesmo `controller timed out` no kernel anterior, descartando a atualização de 2026-07-11 como causa |
| Mouse externo (Logitech, USB/wireless) | Funciona normalmente — não é afetado, confirma que o problema é isolado ao barramento I2C do touchpad |

---

## Status

**Sem solução conhecida.** Pesquisa não encontrou fix documentado específico para esse hardware:

- [Andycodeman/samsung-galaxy-book-linux-fixes](https://github.com/Andycodeman/samsung-galaxy-book-linux-fixes) — tem fixes para webcam/mic/speaker (ver [webcam-setup.md](webcam-setup.md)), nenhum para touchpad.
- [Thread no fórum Arch](https://bbs.archlinux.org/viewtopic.php?id=237288) — caso quase idêntico com touchpad ELAN1200 (também via `i2c_hid_acpi`), sem solução definitiva encontrada pelo autor.

**Workaround:** mouse externo. Para uso sem mouse, às vezes um ciclo de rebind manual (unbind + bind) recupera o touchpad — não é garantido, e mesmo quando "funciona" sem erro de log, o cursor pode continuar sem responder.

```bash
DEV="i2c-IMG4100:00"
echo "$DEV" | sudo tee /sys/bus/i2c/drivers/i2c_hid_acpi/unbind
echo "$DEV" | sudo tee /sys/bus/i2c/drivers/i2c_hid_acpi/bind
```

**Próximo passo:** reportar upstream (Bugzilla do kernel, categoria I2C/HID, ou lista `linux-input`), com a evidência reunida aqui — reproduzível em dois kernels diferentes, mesmo ACPI HID, mesmo controlador, hardware fisicamente funcional.

Ver também [TROUBLESHOOTING.md](../TROUBLESHOOTING.md).
