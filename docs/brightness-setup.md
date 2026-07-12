# Brilho da Tela — Atalhos, Indicador na Waybar e Fix de Kernel (DPCD)

**Hardware:** Samsung 960XFH — Intel Core i9-13900H (Raptor Lake-P / Iris Xe) + NVIDIA GeForce RTX 4070 Max-Q (híbrido, sem MUX ativo — painel eDP conectado à Intel)
**Painel de backlight:** `/sys/class/backlight/intel_backlight`
**Sistema:** Arch Linux, kernel 7.1.3-arch1-2, boot via systemd-boot + UKI

---

## Contexto

Os atalhos físicos de brilho (`Fn+F2` diminuir, `Fn+F3` aumentar) e o indicador na Waybar pareciam funcionar — o percentual mudava normalmente — mas a tela **não ficava fisicamente mais clara ou mais escura**. Não era falta de bind nem de pacote: o valor era escrito corretamente em `/sys/class/backlight/intel_backlight/brightness`, só não chegava a ter efeito no painel.

---

## Diagnóstico

1. **Binds já existiam e estavam corretos.** `Fn+F2`/`Fn+F3` são traduzidos pelo firmware do teclado diretamente para os keysyms padrão `XF86MonBrightnessDown`/`XF86MonBrightnessUp` — não é necessário nenhum bind extra além do que já está em `hyprland.lua`:

   ```lua
   hl.bind("XF86MonBrightnessUp",   hl.dsp.exec_cmd("brightnessctl -n2 set 5%+"), { locked = true, repeating = true })
   hl.bind("XF86MonBrightnessDown", hl.dsp.exec_cmd("brightnessctl -n2 set 5%-"), { locked = true, repeating = true })
   ```

   Confirmado com `hyprctl binds` (os dois binds aparecem registrados) e com `brightnessctl -l` (o valor de `intel_backlight` muda a cada tecla pressionada).

2. **Teste extremo para descartar "mudança sutil demais para notar":**

   ```bash
   brightnessctl set 100%
   brightnessctl set 1%
   brightnessctl set 50%
   ```

   Nenhuma mudança visível na tela em nenhum dos três passos — confirma que o problema é o driver não repassar o valor ao painel, não uma questão de percepção.

3. **GPU híbrida descartada como causa.** Este notebook tem Intel Iris Xe + NVIDIA RTX 4070. Em notebooks com MUX, é comum o painel estar fisicamente ligado à GPU "errada" para fins de backlight. Descartado verificando o status do conector:

   ```bash
   for c in /sys/class/drm/card*-*; do echo "$c: $(cat $c/status 2>/dev/null)"; done
   # card2-eDP-1: connected
   readlink -f /sys/class/drm/card2 | grep -o 'i915\|nvidia'
   # card2 -> driver i915
   ```

   O painel eDP está mesmo ligado à Intel (`i915`), então o `intel_backlight` é a interface certa.

4. **Driver `samsung_galaxybook` também descartado.** O módulo está carregado e expõe `platform-profile`, LED de teclado (`samsung-galaxybook::kbd_backlight`) e sensor da webcam — mas não tem nenhum atributo de brilho de painel. Confirmado em `/sys/devices/platform/SAM0429:00/`.

**Causa raiz:** o painel deste modelo usa controle de brilho via **DPCD/AUX** (canal auxiliar do DisplayPort, como especificado no eDP), e o driver `i915` não habilita esse caminho por padrão — ele decide entre PWM nativo e DPCD com base na tabela VBT do BIOS, que nesse modelo não leva à detecção correta. O atributo `/sys/class/backlight/intel_backlight/type` fica em `raw` (registro nativo) independente da tentativa, então essa checagem sozinha não serve para confirmar se o DPCD está ativo.

---

## Fix — parâmetro de kernel `i915.enable_dpcd_backlight`

Testado em ordem, cada valor exigindo reboot + regeneração da UKI:

| Valor | Significado | Resultado neste hardware |
|---|---|---|
| `1` | Habilita DPCD, driver escolhe a variante (VESA ou Intel) | Sem efeito |
| `2` | Força interface VESA (padrão do eDP spec) | Sem efeito |
| `3` | Força interface proprietária Intel | **Funcionou** |

Sistema usa **UKI via systemd-boot** (`/boot/EFI/Linux/arch-linux.efi`), com a cmdline lida de `/etc/kernel/cmdline`.

```bash
# Editar a cmdline
sudo sed -i 's/$/ i915.enable_dpcd_backlight=3/' /etc/kernel/cmdline

# Conferir
cat /etc/kernel/cmdline

# Regenerar a UKI com o novo parâmetro
sudo mkinitcpio -P

# Reiniciar para aplicar
sudo reboot
```

`/etc/kernel/cmdline` final (relevante):
```
... nvidia_drm.fbdev=1 i915.enable_dpcd_backlight=3
```

> **Dica de diagnóstico:** se `=3` também não funcionar em outro hardware, adicionar `drm.debug=0x1e` na mesma cmdline (junto com o valor de `enable_dpcd_backlight` sendo testado) ativa logs verbosos de driver/KMS/panel, inspecionáveis via `journalctl -k -b | grep -i backlight` sem precisar de sudo para ler. **Remover depois de resolver** — é bem verboso para deixar permanente:
> ```bash
> sudo sed -i 's/ drm.debug=0x1e//' /etc/kernel/cmdline
> sudo mkinitcpio -P
> ```

---

## Indicador na Waybar

Módulo nativo `backlight` do Waybar, adicionado entre `network` e `battery`:

```jsonc
"backlight": {
    "device": "intel_backlight",
    "format": "{icon} {percent}%",
    "format-icons": ["󰃞", "󰃟", "󰃠"],
    "tooltip-format": "Brilho: {percent}%"
}
```

```css
#backlight { color: @text; }
```

O scroll do mouse sobre o ícone já ajusta o brilho — é suporte nativo do módulo `backlight`, sem precisar de `on-scroll-up`/`on-scroll-down`. A cor usa `@text` (`#cad3f5`, branco levemente azulado) em vez de uma cor de destaque, para ficar mais neutro que os outros módulos.

---

## Verificação

```bash
brightnessctl -l
# intel_backlight deve aparecer com Current/Max brightness

brightnessctl set 100%; brightnessctl set 10%
# A tela deve responder visivelmente às duas chamadas
```

Testar `Fn+F2`/`Fn+F3` fisicamente e conferir se o percentual na Waybar acompanha a mudança real de brilho da tela.

---

## Arquivos de configuração

| Arquivo | Propósito |
|---|---|
| `/etc/kernel/cmdline` | Parâmetro `i915.enable_dpcd_backlight=3` |
| `~/.config/hypr/hyprland.lua` | Binds `XF86MonBrightnessUp`/`Down` → `brightnessctl` |
| `~/.config/waybar/config.jsonc` | Módulo `backlight` |
| `~/.config/waybar/style.css` | Cor do `#backlight` |

Ver também [TROUBLESHOOTING.md](../TROUBLESHOOTING.md) e [waybar-setup.md](waybar-setup.md).
