# Configuração do Git

**Versão:** 2.54.0

---

## Contexto

O **Git** é necessário tanto para uso pessoal (controle de versão de projetos)
quanto como dependência para instalar pacotes do AUR via `yay`.

---

## Instalação

```bash
sudo pacman -S git
```

---

## Configuração inicial

```bash
git config --global user.name "seu_usuario"
git config --global user.email "seu@email.com"
git config --global init.defaultBranch main
git config --global core.editor nano
git config --global color.ui auto
```

---

## Verificar configuração

```bash
git config --global --list
```

---

## Comandos essenciais

```bash
# Iniciar repositório
git init

# Clonar repositório
git clone https://url-do-repositorio.git

# Ver status dos arquivos
git status

# Adicionar arquivos ao stage
git add arquivo.txt
git add .          # todos os arquivos

# Criar commit
git commit -m "mensagem do commit"

# Ver histórico
git log --oneline

# Enviar para repositório remoto
git push

# Receber atualizações
git pull
```
