# Instalação do yay — AUR Helper

**Versão:** 13.0.1

---

## Contexto

O **yay** (Yet Another Yogurt) é um helper para o AUR (Arch User Repository) —
o repositório comunitário do Arch Linux que contém milhares de pacotes não
presentes nos repositórios oficiais.

O `pacman` só instala pacotes dos repositórios oficiais. O `yay` estende essa
capacidade, permitindo instalar pacotes do AUR com a mesma simplicidade do
`pacman`.

---

## Pré-requisitos

```bash
sudo pacman -S git base-devel
```

- **git** — para clonar o repositório do yay
- **base-devel** — ferramentas de compilação (gcc, make, etc.) necessárias para
  construir pacotes AUR

---

## Instalação

O yay não está nos repositórios oficiais — precisa ser compilado a partir do AUR:

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

- `makepkg -si` — compila o pacote (`s` instala dependências, `i` instala após compilar)

Após a instalação, o diretório clonado pode ser removido:

```bash
cd ..
rm -rf yay
```

---

## Uso

O `yay` usa a mesma sintaxe do `pacman`:

```bash
# Instalar pacote
yay -S nome-do-pacote

# Atualizar tudo (oficial + AUR)
yay -Syu

# Buscar pacote
yay -Ss nome-do-pacote

# Remover pacote
yay -Rns nome-do-pacote

# Informações sobre pacote
yay -Si nome-do-pacote
```

---

## Diferença entre pacman e yay

| Comando | Repositórios |
|---|---|
| `sudo pacman -S pacote` | Apenas repositórios oficiais |
| `yay -S pacote` | Oficiais + AUR |

> **Nota de segurança:** Pacotes do AUR são mantidos pela comunidade, não pelo
> time do Arch Linux. Antes de instalar, verifique o PKGBUILD do pacote no
> site `aur.archlinux.org`.
