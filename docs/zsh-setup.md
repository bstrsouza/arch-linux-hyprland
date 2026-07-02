# Configuração do Zsh + Starship + Plugins

**Sistema:** Arch Linux, instalação mínima  
**Shell:** Zsh 5.9.1 com Starship 1.25.1

---

## Contexto

O Arch Linux usa `bash` como shell padrão. O **Zsh** é uma alternativa mais moderna, com melhor suporte a plugins, autocompletion e personalização. Combinado com o **Starship** (prompt customizável) e dois plugins de produtividade, resulta em uma experiência de terminal significativamente mais agradável.

---

## Pacotes instalados

```bash
sudo pacman -S zsh starship zsh-autosuggestions zsh-syntax-highlighting
```

| Pacote | Função |
|---|---|
| `zsh` | Shell principal |
| `starship` | Prompt customizável com ícones e informações contextuais |
| `zsh-autosuggestions` | Sugere comandos anteriores enquanto você digita |
| `zsh-syntax-highlighting` | Colore os comandos em tempo real (verde = válido, vermelho = inválido) |

---

## Definir o Zsh como shell padrão

```bash
chsh -s /usr/bin/zsh
```

Requer logout e login para ter efeito.

---

## Configuração — ~/.zshrc

```zsh
# Bindings
bindkey "^[[H"   beginning-of-line
bindkey "^[[1~"  beginning-of-line
bindkey "^[[F"   end-of-line
bindkey "^[[4~"  end-of-line
bindkey "^[[1;5D" backward-word
bindkey "^[[1;5C" forward-word
bindkey "^H"     backward-kill-word
bindkey "^[[3~"  delete-char
bindkey "^[[3;5~" kill-word

# Plugins
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

# PATH
export PATH="$HOME/.local/bin:$PATH"

# Starship prompt
eval "$(starship init zsh)"
```

### Sobre os keybindings

Mapeamentos de teclado para navegação no terminal:

| Binding | Ação |
|---|---|
| `Home` | Ir para o início da linha |
| `End` | Ir para o fim da linha |
| `Ctrl + ←` | Voltar uma palavra |
| `Ctrl + →` | Avançar uma palavra |
| `Ctrl + Backspace` | Apagar palavra anterior |
| `Delete` | Apagar caractere à direita |
| `Ctrl + Delete` | Apagar palavra à direita |

### Sobre os plugins

O `zsh-syntax-highlighting` deve ser sempre o **último plugin** a ser carregado. Carregá-lo antes de outros plugins pode causar comportamento incorreto no highlighting.

---

## Configuração do Starship — ~/.config/starship.toml

O Starship está configurado com o preset de **Nerd Fonts**, que exibe ícones para linguagens, ferramentas, status do git e sistema operacional. Requer uma fonte Nerd Font instalada no terminal.

Para gerar o preset padrão de Nerd Fonts:

```bash
starship preset nerd-font-symbols -o ~/.config/starship.toml
```

---

## Verificação

```bash
# Confirmar shell ativo
echo $SHELL

# Versões instaladas
zsh --version
starship --version

# Recarregar configuração sem abrir novo terminal
source ~/.zshrc
```
