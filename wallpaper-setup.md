# Configuração do Wallpaper

**Ferramenta:** awww (fork do swww)  
**Wallpaper atual:** `/usr/share/hypr/wall0.png`

---

## Contexto

O Hyprland possui um wallpaper padrão próprio controlado pela opção
`force_default_wallpaper`. Para usar um wallpaper específico e ter controle
total sobre a imagem exibida, a opção foi desabilitada e o `awww` assumiu
o controle.

> **Nota:** O pacote se chama `swww` no pacman, mas os binários instalados
> são `awww` e `awww-daemon`.

---

## Instalação

```bash
sudo pacman -S swww
```

Binários instalados:
- `awww-daemon` — processo em background que gerencia o wallpaper
- `awww` — cliente para definir/trocar o wallpaper

---

## Configuração no Hyprland

### Desabilitar o wallpaper padrão

Em `~/.config/hypr/hyprland.lua`:

```lua
misc = {
    force_default_wallpaper = 0,  -- 0 = desabilitado, -1 = aleatório
}
```

### Autostart

```lua
hl.on("hyprland.start", function ()
    hl.exec_cmd("awww-daemon")
    hl.exec_cmd("awww img /usr/share/hypr/wall0.png")
    hl.exec_cmd("waybar > /dev/null 2>&1")
    hl.exec_cmd("swaync")
    hl.exec_cmd("hypridle")
    hl.exec_cmd("wl-paste --type text --watch cliphist store")
    hl.exec_cmd("wl-paste --type image --watch cliphist store")
    hl.exec_cmd("/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1")
end)
```

O `awww-daemon` precisa iniciar **antes** do comando `awww img`, pois o
daemon é o processo que efetivamente renderiza o wallpaper.

---

## Wallpapers padrão do Hyprland

Ficam em `/usr/share/hypr/`:

```
wall0.png  ← em uso
wall1.png
wall2.png
```

## Trocar o wallpaper manualmente

```bash
awww img /caminho/para/imagem.png
```

O awww suporta transições animadas ao trocar de wallpaper:

```bash
# Com transição fade
awww img /caminho/para/imagem.png --transition-type fade

# Com transição wipe
awww img /caminho/para/imagem.png --transition-type wipe
```

Para trocar permanentemente, atualize o caminho no autostart do
`~/.config/hypr/hyprland.lua`.
