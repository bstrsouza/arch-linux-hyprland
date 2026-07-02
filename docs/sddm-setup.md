# Configuração do Display Manager — SDDM + Catppuccin Macchiato

**Sistema:** Arch Linux, instalação mínima  
**Tema:** catppuccin-macchiato-blue  
**Objetivo:** Iniciar o Hyprland automaticamente com tela de login gráfica

---

## Contexto

Uma instalação mínima do Arch Linux não inclui interface gráfica de login — o sistema inicia direto no TTY, exigindo login manual e execução do comando `Hyprland` a cada boot.

A solução é instalar um **display manager**: um serviço que inicializa com o sistema, exibe uma tela de login gráfica e inicia o compositor (Hyprland) após a autenticação.

O **SDDM** (Simple Desktop Display Manager) foi escolhido por ser leve, amplamente suportado e compatível com Wayland/Hyprland.

---

## Passo 1 — Instalar o SDDM

```bash
sudo pacman -S sddm
```

---

## Passo 2 — Habilitar o serviço

```bash
sudo systemctl enable sddm
```

O SDDM detecta automaticamente as sessões disponíveis (incluindo Hyprland) através dos arquivos `.desktop` em `/usr/share/wayland-sessions/`.

---

## Passo 3 — Instalar o tema Catppuccin Macchiato

```bash
yay -S catppuccin-sddm-theme-macchiato
```

Instala os temas em `/usr/share/sddm/themes/catppuccin-macchiato-*/`. Variantes disponíveis por accent: blue, mauve, green, lavender, red, etc.

---

## Passo 4 — Configurar o tema

**`/etc/sddm.conf`:**

```ini
[Theme]
Current=catppuccin-macchiato-blue
```

> **Atenção:** O nome deve ser exatamente igual ao diretório em `/usr/share/sddm/themes/`. O SDDM diferencia maiúsculas de minúsculas.

---

## Passo 5 — Reiniciar o SDDM

```bash
sudo systemctl restart sddm
```

---

## Verificação

```bash
cat /etc/sddm.conf

# Listar temas instalados
ls /usr/share/sddm/themes/ | grep catppuccin
```
