# Configuração da Webcam — Intel IPU6 + OV02C10 (Raptor Lake)

**Hardware:** Samsung 960XFH — Intel Core i9-13900H (Raptor Lake)
**Câmera:** Sensor OmniVision OV02C10 (ACPI HID `OVTI02C1`) via Intel IPU6 (`8086:a75d`) + IVSC (Intel Visual Sensing Controller)
**Sistema:** Arch Linux, kernel 7.1.2-arch3-1

---

## Contexto

A webcam interna não funciona out-of-the-box. Não é falta de driver — é uma combinação de problemas conhecidos nessa geração de câmera MIPI da Intel (IPU6), documentados pela comunidade em [Andycodeman/samsung-galaxy-book-linux-fixes](https://github.com/Andycodeman/samsung-galaxy-book-linux-fixes). Essa combinação de sensor com Raptor Lake tem suporte **parcial**: dá pra deixar funcional para videochamadas, mas a qualidade de imagem final é ruim e não tem correção via software — ver [Limitação conhecida](#️-limitação-conhecida--qualidade-de-imagem) no fim deste documento.

---

## Diagnóstico

```bash
journalctl -k -b | grep -i ov02c10
```

```
intel-ipu6 0000:00:05.0: Found supported sensor OVTI02C1:00
intel-ipu6 0000:00:05.0: Connected 1 cameras
ov02c10 i2c-OVTI02C1:00: error -EINVAL: external clock 26000000 is not supported
ov02c10 i2c-OVTI02C1:00: probe with driver ov02c10 failed with error -22
```

**Causa:** o driver `ov02c10` do kernel só aceita clock externo de 19.2 MHz. Este notebook (assim como outros Samsung Galaxy Book com IPU6 Raptor Lake) reporta 26 MHz via ACPI, e o driver rejeita a inicialização do sensor. Isso não é um problema de permissão nem de pacote faltando — o sensor nunca chega a inicializar.

Além disso, essa câmera vem montada de cabeça para baixo (BIOS não reporta a rotação corretamente), e o `libcamera` não tem uma calibração própria para o OV02C10.

---

## Passo 1 — Corrigir o clock do sensor (patch DKMS)

```bash
curl -sL https://github.com/Andycodeman/samsung-galaxy-book-linux-fixes/archive/refs/heads/main.tar.gz | tar xz
cd samsung-galaxy-book-linux-fixes-main/ov02c10-26mhz-fix
sudo bash install.sh
sudo reboot
```

**Por quê:** instala uma cópia do driver `ov02c10.c` original da Intel com uma única mudança — aceitar clock de 26 MHz além de 19.2 MHz (mesmo patch [proposto na lista linux-media](https://lore.kernel.org/linux-media/CAKP_te-WT+HTEyhSvQ3snEOaTp5B1OUL18JjuzO238=_fTOuXQ@mail.gmail.com/) e ainda não mergeado upstream). Empacotado via DKMS, então sobrevive a atualizações de kernel.

**Verificação pós-reboot:**

```bash
sudo dmesg | grep -i ov02c10
# Não deve mais aparecer "external clock 26000000 is not supported"
modinfo ov02c10 | grep filename
# Caminho deve conter /updates/
```

---

## Passo 2 — Instalar a pilha de câmera (libcamera + PipeWire + camera-relay)

```bash
cd ../webcam-fix-libcamera
./install.sh
sudo reboot
```

**Por quê:** este é o instalador recomendado pelo próprio projeto (em vez do `webcam-fix/`, que usa a HAL proprietária da Intel — ver [Tentativa abandonada](#tentativa-abandonada--pilha-proprietária-da-intel) abaixo). Ele:

- Detecta e aplica automaticamente o fix de rotação (patch DKMS `ipu-bridge`, necessário porque o 960XFH está na lista de modelos com sensor invertido)
- Compila o `libcamera` a partir do código-fonte em `/usr/local` (necessário no Arch: o pacote oficial `libcamera` 0.7.1 tem os símbolos do sensor OV02C10 mas não os registra em runtime — bug conhecido do Arch/CachyOS, senão a imagem fica preta)
- Integra com PipeWire/WirePlumber
- Instala o `camera-relay` — uma ponte v4l2loopback sob demanda para apps que não falam PipeWire nativamente (Zoom, VLC, e o próprio Chromium)

### Bugs encontrados no instalador (nesta versão do repositório)

| Bug | Correção |
|---|---|
| URL do libcamera (`git.libcamera.org`) migrou para `gitlab.freedesktop.org` — `git clone` recusa o redirect entre hosts | Editar `install.sh`, trocar a URL na função `build_libcamera_from_source()` |
| `LIBCAMERA_IPA_MODULE_PATH` nunca é configurado (`/etc/environment.d/libcamera-ipa.conf` não é criado) — a detecção de diretório procura `.so` um nível acima de onde o `meson` realmente instala | Criar o arquivo manualmente (Passo 3) |
| Mesmo bug de path dentro do script `/usr/local/bin/camera-relay` (função `detect_ipa_path`, por volta da linha 276) — `echo "$path"` deveria ser `echo "$path/ipa"` quando o `.so` está numa subpasta `ipa/` | `sudo sed -i '276s/echo "\$path"/echo "\$path\/ipa"/' /usr/local/bin/camera-relay` |

Se reinstalar do zero, esses três bugs provavelmente ainda estarão presentes (é a versão do repositório no momento em que este documento foi escrito).

---

## Passo 3 — Corrigir a variável de ambiente do IPA

```bash
sudo tee /etc/environment.d/libcamera-ipa.conf > /dev/null << 'EOF'
LIBCAMERA_IPA_MODULE_PATH=/usr/local/lib/libcamera/ipa
GST_PLUGIN_PATH=/usr/local/lib/gstreamer-1.0
EOF
```

**Por quê:** sem isso, o `libcamera` carrega o módulo IPA (algoritmo de exposição/AWB) do pacote do sistema, que não tem o sensor helper do OV02C10 registrado — a câmera aparece na lista mas a imagem fica completamente preta (ganho travado no genérico).

Depois de criar o arquivo, reinicie a sessão (ou pelo menos `systemctl --user restart camera-relay pipewire wireplumber`) para os serviços pegarem a variável nova.

---

## Passo 4 — Ajuste de exposição e cor (tuning manual)

A tuning file padrão que o instalador entrega (`/usr/local/share/libcamera/ipa/simple/ov02c10.yaml`) deixa a imagem escura e com viés de cor roxo/magenta. Versão final usada, depois de comparar várias combinações lado a lado numa videochamada:

```bash
sudo tee /usr/local/share/libcamera/ipa/simple/ov02c10.yaml > /dev/null << 'EOF'
# SPDX-License-Identifier: CC0-1.0
%YAML 1.1
---
version: 1
algorithms:
  - BlackLevel:
      blackLevel: 1024
  - Awb:
  - Ccm:
      ccms:
        - ct: 5000
          ccm: [ 1.20, 0.00, -0.20,
                 0.10, 0.80, 0.10,
                -0.20, 0.00, 1.20 ]
  - Adjust:
  - Agc:
...
EOF
systemctl --user restart camera-relay.service pipewire wireplumber
```

**Por quê cada mudança:**

| Parâmetro | Padrão do instalador | Usado aqui | Motivo |
|---|---|---|---|
| `blackLevel` | ~4096 (implícito, alto demais para esse sensor) | `1024` | Valor alto crushava a imagem (escura); `0` (sem correção nenhuma) deixava a imagem "enevoada"/lavada. `1024` é o meio-termo que preserva contraste sem escurecer demais. |
| `Ccm` (matriz de cor) | Conservadora (`1.05 / 0.92 / 1.05` na diagonal) | "Anti-green medium" (`1.20 / 0.80 / 1.20`) | O sensor lê com viés esverdeado/roxo nessa iluminação; reduzir o canal verde ~20% resolve. Presets mais agressivos ("anti-green strong") geraram um split de cor (pele quente, parede fria). |
| `Adjust` | Ausente | Presente (vazio) | Ativa o estágio de contraste automático via histograma do `libcamera` 0.7+, ausente na tuning file padrão. |

Para testar outros presets de cor interativamente (com preview ao vivo), o repositório inclui uma ferramenta pronta:

```bash
cd samsung-galaxy-book-linux-fixes-main/webcam-fix-libcamera
./tune-ccm.sh
```

---

## Verificação

```bash
cam --list
# Deve mostrar: 1: Internal front camera (\_SB_.PC00.LNK0)

gst-launch-1.0 libcamerasrc ! videoconvert ! autovideosink
# Preview ao vivo, direto via libcamera/PipeWire

mpv av://v4l2:/dev/video0 --profile=low-latency --untimed --no-correct-pts
# Preview via Camera Relay (v4l2), o que Zoom/VLC/Chromium efetivamente usam
```

No Firefox/Chromium, ao permitir o acesso à câmera, selecionar explicitamente **"Camera Relay"** na lista (não uma das ~30 entradas cruas "Intel IPU6 ISYS Capture N" — essas são nós internos do pipeline, inúteis para captura direta).

---

## ⚠️ Limitação conhecida — qualidade de imagem

Mesmo depois de todos os ajustes acima, a imagem final tem qualidade ruim — aparência "enevoada"/suave, lembrando uma câmera analógica antiga (baixo contraste local, falta de nitidez), bem inferior a uma webcam USB comum ou à câmera de um celular.

**Causa raiz (não é bug de configuração):**

- O ganho analógico do sensor fica preso perto do máximo (15.5x de um teto de 15.5x) mesmo com bastante luz — isso sozinho já gera ruído visível.
- O `libcamera` não tem uma calibração própria para o OV02C10 (confirmado na [issue oficial do projeto](https://github.com/intel/ipu6-drivers/issues/431)) — só uma tuning genérica feita pela comunidade.
- A pilha "Software ISP" do `libcamera` (usada aqui) não tem nitidez/realce de contraste local como um ISP de hardware de verdade — e não há suporte a ISP de hardware para essa combinação no Linux mainline.

Ajustes de `blackLevel`/CCM melhoram brilho e cor de forma mensurável (~+44% de brilho, correção de viés de cor), mas não mudam esse caráter geral da imagem — não é algo resolvível via tuning YAML.

**Status:** aceito como está. Câmera funcional para videochamadas, qualidade abaixo do ideal. Se a qualidade de imagem for essencial, uma webcam USB externa é a alternativa prática.

---

## Tentativa abandonada — pilha proprietária da Intel

Como alternativa à pilha open-source acima, foi testada a pilha proprietária da Intel (`icamerasrc` + `v4l2-relayd`, compilada do zero a partir de `intel/ipu6-camera-bins`, `intel/ipu6-camera-hal` e `intel/icamerasrc`), na expectativa de que a calibração oficial da Intel desse um resultado melhor.

**Motivo do abandono:** depois de corrigir uma série de problemas de build (CMake muito novo rejeitando o projeto, `-Werror` incompatível com GCC atual, bibliotecas estáticas faltando no script de instalação, `v4l2-relayd` usando autotools em vez do meson que o script assumia, arquivo `/etc/default/v4l2-relayd` obrigatório nunca criado, pacote `gst-plugins-good` faltando), a HAL da Intel (`libcamhal`) rejeita **todas** as resoluções testadas (1920x1080, 1920x1088, 1928x1092) com `Stream config is not supported`, mesmo elas constando como suportadas nos três arquivos de configuração do sensor disponíveis (`OV02C10_1BG203N3_ADL`, `OV02C10_1SG204N3_ADL`, `OV02C10_CIFME14_ADL`). É uma inconsistência interna entre o que a HAL declara suportar e o que a validação de stream aceita em runtime — resolver isso exigiria engenharia reversa da lógica interna de "graph config" da Intel, sem acesso a documentação/suporte oficial para essa combinação específica de sensor+plataforma.

A pilha proprietária foi completamente desinstalada e revertida para a pilha libcamera descrita acima.

---

## Arquivos de configuração criados

| Arquivo | Propósito |
|---|---|
| `/etc/modules-load.d/ivsc.conf` | Carrega os módulos IVSC no boot |
| `/etc/modprobe.d/ivsc-camera.conf` | Softdep: IVSC carrega antes do sensor |
| `/etc/udev/rules.d/90-hide-ipu6-v4l2.rules` | Esconde os ~30 nós `/dev/video*` crus do IPU6 |
| `/etc/wireplumber/wireplumber.conf.d/50-disable-ipu6-v4l2.conf` | Esconde os mesmos nós no PipeWire |
| `/etc/environment.d/libcamera-ipa.conf` | Caminho do módulo IPA (fix manual, Passo 3) |
| `/etc/environment.d/10-libcamera-softisp.conf` | Força debayer por CPU (GPU NVIDIA detectada) |
| `/usr/local/share/libcamera/ipa/simple/ov02c10.yaml` | Tuning de exposição/cor (Passo 4) — **sobrescrito a cada reinstalação** |
| `/usr/local/bin/camera-relay` + `camera-relay-monitor` | Ponte v4l2loopback sob demanda |
| `/etc/modprobe.d/99-camera-relay-loopback.conf` | Configuração do `v4l2loopback` para o relay |

Módulos DKMS instalados (`dkms status`): `ov02c10/1.0` (clock 26MHz), `ipu-bridge-fix/1.4` (rotação), `v4l2loopback/0.15.4`.

Ver também [TROUBLESHOOTING.md](../TROUBLESHOOTING.md).
