# Configuração do Polkit Agent — polkit-gnome

---

## Contexto

O **polkit** é o sistema do Linux responsável por autorizar ações privilegiadas
em aplicativos gráficos. Quando um app precisa de permissão de root — como
formatar um disco no Thunar, instalar uma impressora ou modificar configurações
de rede — ele solicita autenticação via polkit.

Sem um **agente polkit** rodando, a janela de senha simplesmente não aparece e
a ação falha silenciosamente.

O `polkit-gnome` é o agente mais compatível e amplamente usado em ambientes
Wayland não-GNOME, como o Hyprland.

---

## Instalação

```bash
sudo pacman -S polkit-gnome
```

---

## Autostart

Em `~/.config/hypr/hyprland.lua`:

```lua
hl.exec_cmd("/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1")
```

O agente é iniciado junto com o Hyprland e fica em segundo plano aguardando
solicitações de autenticação.

---

## Como funciona

1. App gráfico solicita uma ação privilegiada
2. O polkit intercepta a solicitação
3. O agente exibe uma janela pedindo a senha do usuário
4. Após autenticação, a ação é executada com privilégio elevado

---

## Teste sem reiniciar

```bash
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
```
