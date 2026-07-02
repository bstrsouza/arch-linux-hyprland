# Configuração do Driver NVIDIA — Arch Linux + Hyprland

**Hardware:** Samsung 960XFH — Intel Core i9-13900H + NVIDIA RTX 4070 Mobile (Max-Q)  
**Sistema:** Arch Linux, Hyprland 0.55.4, kernel 7.0.12-arch1-1  
**Bootloader:** systemd-boot com UKI (Unified Kernel Image)

---

## Contexto

O sistema saiu de fábrica usando o driver `nouveau` para a RTX 4070. O `nouveau` é um driver open-source criado por engenharia reversa, sem suporte oficial da NVIDIA. Para GPUs da arquitetura Ada Lovelace (RTX 40xx), ele não oferece aceleração 3D real e pode causar instabilidade no Hyprland.

A solução é instalar o `nvidia-open`, que são os módulos de kernel open-source **oficiais** lançados pela própria NVIDIA a partir de 2022. São a opção recomendada para GPUs Turing em diante.

Este laptop usa **gráficos híbridos (NVIDIA Optimus)**: a Intel Iris Xe gerencia o display e o consumo do dia a dia, enquanto a RTX 4070 é acionada sob demanda para tarefas pesadas (jogos, renders, etc). Essa arquitetura economiza bateria sem abrir mão da performance quando necessário.

---

## Passo 1 — Instalar o driver

```bash
sudo pacman -S nvidia-open nvidia-utils lib32-nvidia-utils
```

**Por quê:**
- `nvidia-open` — módulos de kernel que substituem o `nouveau`
- `nvidia-utils` — utilitários do espaço de usuário (`nvidia-smi`, OpenGL, Vulkan)
- `lib32-nvidia-utils` — suporte a aplicações 32-bit (necessário para jogos via Steam/Wine/Proton)

---

## Passo 2 — Blacklist do nouveau

```bash
sudo tee /etc/modprobe.d/blacklist-nouveau.conf << 'EOF'
blacklist nouveau
options nouveau modeset=0
EOF
```

**Por quê:** Mesmo com o `nvidia-open` instalado, o kernel pode carregar o `nouveau` por padrão, causando conflito. Este arquivo instrui o kernel a nunca carregar o módulo `nouveau`, garantindo que apenas o driver correto seja usado.

---

## Passo 3 — Configurar o mkinitcpio

Edite `/etc/mkinitcpio.conf`:

```bash
sudo nano /etc/mkinitcpio.conf
```

Altere as linhas `MODULES` e `HOOKS`:

```
# Antes
MODULES=()
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)

# Depois
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
HOOKS=(base udev autodetect microcode modconf keyboard keymap consolefont block filesystems fsck)
```

**Por quê — `MODULES`:** Os quatro módulos NVIDIA precisam ser carregados no initramfs (o ambiente mínimo que inicia antes do sistema de arquivos principal ser montado). Isso garante que o driver esteja disponível desde o início do boot, evitando uma tela preta ao iniciar o Hyprland.

| Módulo | Função |
|---|---|
| `nvidia` | Módulo principal do driver |
| `nvidia_modeset` | Gerenciamento do modo de vídeo (KMS) |
| `nvidia_uvm` | Memória unificada (usado por CUDA e Vulkan) |
| `nvidia_drm` | Integração com o subsistema DRM do kernel (necessário para Wayland) |

**Por quê — remover `kms`:** O hook `kms` (Kernel Mode Setting) carrega os drivers de vídeo genéricos do kernel no initramfs. Com o driver NVIDIA, isso causa conflito porque o `kms` tenta inicializar o display antes do driver proprietário estar pronto. Manter os dois resulta em falha no boot ou tela preta.

---

## Passo 4 — Adicionar parâmetros do kernel

> **Nota sobre o bootloader:** Este sistema usa systemd-boot com UKI (Unified Kernel Image). Em vez de arquivos `.conf` separados por entrada de boot, os parâmetros do kernel ficam centralizados em `/etc/kernel/cmdline`. A imagem gerada é `/boot/EFI/Linux/arch-linux.efi`.

Edite o arquivo de parâmetros:

```bash
sudo nano /etc/kernel/cmdline
```

Adicione ao final da linha existente (sem quebrar em nova linha):

```
nvidia_drm.modeset=1 nvidia_drm.fbdev=1
```

O arquivo completo deve ficar assim:

```
root=PARTUUID=<SEU-PARTUUID> zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs nvidia_drm.modeset=1 nvidia_drm.fbdev=1
```

**Por quê:**
- `nvidia_drm.modeset=1` — ativa o Kernel Mode Setting do driver NVIDIA, obrigatório para Wayland. Sem isso o Hyprland não consegue iniciar.
- `nvidia_drm.fbdev=1` — ativa o framebuffer do driver NVIDIA, necessário a partir do kernel 6.x para exibir o console e o splash de boot corretamente.

---

## Passo 5 — Reconstruir a imagem UKI

```bash
sudo mkinitcpio -P
```

**Por quê:** As alterações nos passos 3 e 4 não têm efeito imediato — elas definem como a próxima imagem de boot será gerada. Este comando reconstrói o arquivo `/boot/EFI/Linux/arch-linux.efi` incorporando os novos módulos e parâmetros. Só depois disso as mudanças entram em vigor no próximo boot.

> O `-P` reconstrói todos os presets definidos em `/etc/mkinitcpio.d/`. No caso deste sistema, isso gera `arch-linux.efi` e `arch-linux-fallback.efi`.

---

## Passo 6 — Configurar variáveis de ambiente no Hyprland

Adicione as seguintes linhas na seção `ENVIRONMENT VARIABLES` do `~/.config/hypr/hyprland.lua`:

```lua
-- NVIDIA (hybrid graphics: Intel display + NVIDIA via PRIME offloading)
hl.env("LIBVA_DRIVER_NAME", "nvidia")
hl.env("GBM_BACKEND", "nvidia-drm")
hl.env("__GLX_VENDOR_LIBRARY_NAME", "nvidia")
hl.env("NVD_BACKEND", "direct")
```

**Por quê:**

| Variável | Motivo |
|---|---|
| `LIBVA_DRIVER_NAME=nvidia` | Direciona a aceleração de vídeo (VA-API) para usar a NVIDIA, habilitando decodificação de vídeo por hardware |
| `GBM_BACKEND=nvidia-drm` | Define o backend GBM (Generic Buffer Manager) para o driver DRM da NVIDIA, necessário para o Wayland funcionar corretamente |
| `__GLX_VENDOR_LIBRARY_NAME=nvidia` | Força o uso das bibliotecas GLX da NVIDIA em vez das genéricas do Mesa, evitando conflito em aplicações XWayland |
| `NVD_BACKEND=direct` | Ativa o backend de decodificação de vídeo direto da NVIDIA (mais eficiente que o legado) |

---

## Passo 7 — Reiniciar

```bash
sudo reboot
```

---

## Verificação pós-instalação

Após reiniciar, confirme que o driver está ativo:

```bash
# Verificar driver em uso por cada GPU
lspci -k | grep -A 2 "VGA\|3D"

# Listar módulos NVIDIA carregados
lsmod | grep nvidia

# Informações da GPU via driver NVIDIA
nvidia-smi
```

Na saída do `lspci`, a RTX 4070 deve mostrar `Kernel driver in use: nvidia` (não `nouveau`).

### Resultado obtido (22/06/2026)

```
# lspci -k | grep -A 2 "VGA\|3D"
01:00.0 3D controller: NVIDIA GeForce RTX 4070 Max-Q / Mobile
    Kernel driver in use: nvidia          ✓

# lsmod | grep nvidia
nvidia_drm        167936  1
nvidia_uvm       2449408  0
nvidia_modeset   1929216  1 nvidia_drm
nvidia          18190336  7 nvidia_uvm,nvidia_modeset   ✓

# nvidia-smi
Driver: 610.43.02 | Temp: 36°C | Pwr: 10W/30W | VRAM: 1MiB/8188MiB
Disp.A: Off  →  Hyprland rodando na Intel (comportamento correto para hybrid graphics)  ✓
```

---

## Uso da NVIDIA sob demanda (PRIME Offloading)

O Hyprland continua rodando na Intel Iris Xe por padrão — o que é o comportamento correto para um laptop, pois consome menos energia. Para executar um aplicativo específico na RTX 4070:

```bash
# Método manual
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia nome-do-app

# Com o pacote nvidia-prime (mais conveniente)
sudo pacman -S nvidia-prime
prime-run nome-do-app
```

---

## Impacto no Dual Boot com Windows

Nenhuma das etapas acima afeta a instalação do Windows. A partição EFI (`/dev/nvme0n1p1`) contém pastas separadas para cada sistema:

```
/boot/EFI/Linux/       ← arquivos do Arch Linux (modificados pelo mkinitcpio)
/boot/EFI/Microsoft/   ← arquivos do Windows   (intocados)
```

O Windows possui seu próprio driver NVIDIA (GeForce Game Ready Driver), completamente independente. Os dois coexistem sem conflito.
