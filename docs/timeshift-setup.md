# Configuração de Backup — Timeshift

**Versão:** 25.12.4  
**Modo:** BTRFS  
**Partição:** nvme0n1p4 (342G, montada em `/` e `/home`)

---

## Contexto

O **Timeshift** é uma ferramenta de snapshots do sistema, similar ao "Ponto de Restauração" do Windows. No modo BTRFS, os snapshots são nativos do filesystem — extremamente rápidos e eficientes, armazenando apenas as diferenças entre estados.

---

## Instalação

```bash
sudo pacman -S timeshift
```

Para abrir:
```bash
sudo timeshift-launcher
```

---

## Configuração

**Modo:** BTRFS (detectado automaticamente)  
**Subvolumes incluídos:** `@` (sistema) e `@home` (arquivos pessoais e configs)

### Agendamento de snapshots

| Nível | Quantidade mantida |
|---|---|
| Boot | 5 |
| Hourly | 0 |
| Daily | 5 |
| Weekly | 3 |
| Monthly | 2 |

O Timeshift mantém apenas a quantidade configurada — quando o limite é atingido, o snapshot mais antigo é removido automaticamente.

---

## Por que BTRFS é melhor que RSYNC

| | BTRFS | RSYNC |
|---|---|---|
| Velocidade | Instantâneo | Minutos/horas |
| Espaço | Incremental (só o que mudou) | Cópia completa |
| Restauração | Muito rápida | Lenta |
| Requisito | Filesystem BTRFS | Qualquer filesystem |

---

## Melhores práticas

### Antes de atualizações do sistema
Sempre crie um snapshot manual antes de atualizar pacotes importantes:

```bash
sudo timeshift --create --comments "antes de atualizar sistema"
sudo pacman -Syu
```

Se a atualização quebrar algo, restaure o snapshot anterior.

### Antes de modificar configurações críticas
Crie um snapshot antes de editar arquivos como:
- `~/.config/hypr/hyprland.lua`
- `/etc/mkinitcpio.conf`
- `/etc/kernel/cmdline`
- Drivers NVIDIA

### Após configurações bem-sucedidas
Crie um snapshot como marco:
```bash
sudo timeshift --create --comments "sistema configurado e estável"
```

### Verificar snapshots existentes
```bash
sudo timeshift --list
```

### Restaurar um snapshot (linha de comando)
```bash
sudo timeshift --restore
```

### Frequência de verificação
Não é necessário monitorar constantemente. Abra o Timeshift apenas:

1. **Antes de atualizações grandes** — criar snapshot manual
2. **Quando algo quebrar** — restaurar snapshot anterior
3. **Mensalmente** — conferir se os snapshots automáticos estão sendo criados

---

## O que o Timeshift NÃO substitui

- **Backup de arquivos pessoais importantes** (documentos, fotos) em disco externo ou nuvem — o Timeshift é para restaurar o sistema, não para recuperar arquivos deletados acidentalmente
- **Backup offsite** — snapshots ficam no mesmo disco, se o disco falhar os snapshots são perdidos junto

---

## Comandos úteis

```bash
# Criar snapshot manual com descrição
sudo timeshift --create --comments "descrição"

# Listar todos os snapshots
sudo timeshift --list

# Restaurar interativamente
sudo timeshift --restore

# Deletar um snapshot específico
sudo timeshift --delete --snapshot '2026-06-22_00-00-01'
```
