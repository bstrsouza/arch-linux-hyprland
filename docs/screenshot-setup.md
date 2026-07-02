# Configuração de Screenshot — hyprshot

**Versão:** 1.3.0  
**Pacote:** hyprshot (repositório oficial Arch)  
**Ecossistema:** Hyprland nativo

---

## Contexto

O **hyprshot** é a ferramenta de captura de tela nativa do Hyprland. Suporta seleção de região, janela ou monitor inteiro, salva o arquivo em disco e copia automaticamente para o clipboard.

---

## Instalação

```bash
sudo pacman -S hyprshot
```

---

## Diretório de saída

```bash
mkdir -p ~/Pictures/Screenshots
```

---

## Atalhos configurados

Em `~/.config/hypr/hyprland.lua`:

```lua
hl.bind("Print",               hl.dsp.exec_cmd("hyprshot -m region -o ~/Pictures/Screenshots"))
hl.bind(mainMod .. " + Print", hl.dsp.exec_cmd("hyprshot -m window -o ~/Pictures/Screenshots"))
hl.bind("SHIFT + Print",       hl.dsp.exec_cmd("hyprshot -m output -o ~/Pictures/Screenshots"))
```

| Tecla | Ação |
|---|---|
| `Print` | Selecionar região com o mouse |
| `Super + Print` | Capturar janela ativa |
| `Shift + Print` | Capturar monitor inteiro |

---

## Modos disponíveis

| Modo | Descrição |
|---|---|
| `region` | Seleção livre com o mouse |
| `window` | Janela clicada ou ativa |
| `output` | Monitor inteiro |
| `active` | Janela ou monitor atualmente em foco |

---

## Opções úteis

| Opção | Descrição |
|---|---|
| `-o <pasta>` | Diretório de saída |
| `--clipboard-only` | Copia para clipboard sem salvar arquivo |
| `-z` | Congela a tela ao selecionar região |
| `-s` | Silencioso (sem notificação) |
