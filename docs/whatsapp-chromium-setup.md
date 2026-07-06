# Duas contas de WhatsApp — Chromium

**Atalhos:** `Super + W` (WA Personal) · `Super + Shift + W` (WA Work)
**Workspace:** ambas as janelas sempre abrem no workspace 9

---

## Contexto

Para usar duas contas pessoais de WhatsApp Web ao mesmo tempo, cada uma roda como uma janela de app independente do **Chromium** (`--app=`, sem abas nem barra de endereço), com um `--user-data-dir` próprio por conta. Isso garante perfis totalmente isolados — sessão, cookies e login de uma conta não têm nenhum contato com a outra — sem depender do seletor de "Perfil 1/2" do Chromium nem de login com conta Google.

---

## Instalação

```bash
sudo pacman -S chromium
```

---

## Perfis isolados

Cada conta usa seu próprio diretório de dados:

```bash
mkdir -p ~/.config/chromium-whatsapp1 ~/.config/chromium-whatsapp2
```

Comando de lançamento de cada conta:

```bash
chromium --app=https://web.whatsapp.com --user-data-dir=~/.config/chromium-whatsapp1 --profile-directory=Personal
chromium --app=https://web.whatsapp.com --user-data-dir=~/.config/chromium-whatsapp2 --profile-directory=Work
```

> **Nota sobre `--class`:** a flag `--class=` do Chromium é **ignorada** quando combinada com `--app=` — o WM_CLASS real da janela é derivado do host da URL + `--profile-directory`, no formato `chrome-<host>-<profile-directory>`. Por isso os dois perfis usam `--profile-directory` diferente (`Personal` / `Work`): sem isso, as duas janelas teriam a mesma classe (`chrome-web.whatsapp.com__-Default`) e não daria para diferenciá-las nas regras do Hyprland. Confirmado testando com `hyprctl clients` — as classes reais são:
> - `chrome-web.whatsapp.com__-Personal`
> - `chrome-web.whatsapp.com__-Work`

No primeiro lançamento de cada conta, faça login escaneando o QR code do WhatsApp Web. A sessão fica salva no respectivo `--user-data-dir` e persiste entre reinicializações.

---

## Configuração no Hyprland

Em `~/.config/hypr/hyprland.lua`, seção `KEYBINDINGS`:

```lua
local whatsapp1Cmd = "chromium --app=https://web.whatsapp.com --force-dark-mode --user-data-dir=" .. os.getenv("HOME") .. "/.config/chromium-whatsapp1 --profile-directory=Personal"
local whatsapp2Cmd = "chromium --app=https://web.whatsapp.com --force-dark-mode --user-data-dir=" .. os.getenv("HOME") .. "/.config/chromium-whatsapp2 --profile-directory=Work"

hl.bind(mainMod .. " + W",         hl.dsp.exec_cmd(whatsapp1Cmd))
hl.bind(mainMod .. " + SHIFT + W", hl.dsp.exec_cmd(whatsapp2Cmd))
```

`--force-dark-mode` força o tema escuro nos elementos nativos do Chromium (menu de contexto, diálogos, barra de busca). O conteúdo da própria página (`web.whatsapp.com`) já recebe `prefers-color-scheme: dark` a partir do ajuste de dark mode do sistema (seção abaixo) — mas o WhatsApp Web só fica escuro de fato se a opção de tema dentro do próprio app estiver em **Dark** ou **System default** (Configurações → Tema, dentro do WhatsApp Web já logado).

Seção `WINDOWS AND WORKSPACES`, para forçar as duas janelas a sempre abrirem no workspace 9 sem roubar o foco de onde você estiver (`silent`):

```lua
hl.window_rule({
    name      = "whatsapp1-workspace",
    match     = { class = "^chrome-web\\.whatsapp\\.com__-Personal$" },
    workspace = "9 silent",
})

hl.window_rule({
    name      = "whatsapp2-workspace",
    match     = { class = "^chrome-web\\.whatsapp\\.com__-Work$" },
    workspace = "9 silent",
})
```

---

## Entradas no wofi

Dois arquivos `.desktop` em `~/.local/share/applications/`, para aparecerem no launcher (`Super + R`) como **WA Personal** e **WA Work**:

**`whatsapp1.desktop`:**
```ini
[Desktop Entry]
Type=Application
Name=WA Personal
Comment=WhatsApp Web - conta pessoal
Exec=chromium --app=https://web.whatsapp.com --force-dark-mode --user-data-dir=/home/bruno/.config/chromium-whatsapp1 --profile-directory=Personal
Icon=whatsapp
Terminal=false
StartupWMClass=chrome-web.whatsapp.com__-Personal
Categories=Network;InstantMessaging;
```

**`whatsapp2.desktop`:** mesma estrutura, trocando `Name=WA Work`, `whatsapp2`/`Work` e `StartupWMClass=chrome-web.whatsapp.com__-Work`.

> `Exec=` não passa por um shell nem expande `~`/`$HOME`/`%h` (field code inválido conforme a especificação freedesktop) — use o caminho absoluto.

Ícone usado: SVG oficial do WhatsApp salvo em `~/.local/share/icons/hicolor/scalable/apps/whatsapp.svg`, referenciado como `Icon=whatsapp` (resolvido via tema de ícones hicolor).
