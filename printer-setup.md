# Configuração de Impressora — CUPS

---

## Contexto

O **CUPS** (Common Unix Printing System) é o sistema de impressão padrão do
Linux. Suporta impressoras USB, rede Wi-Fi e servidores de impressão
compartilhados.

---

## Instalação

```bash
sudo pacman -S cups cups-pdf ghostscript gsfonts
yay -S samsung-unified-driver-printer
sudo systemctl enable --now cups
```

---

## Impressora USB (Samsung — uso doméstico)

O CUPS detecta e configura automaticamente impressoras USB ao conectá-las
com o serviço ativo.

**Passos:**
1. Ligue a impressora
2. Conecte o cabo USB
3. Abra o gerenciador: `system-config-printer`
4. A impressora aparece na lista automaticamente

**Verificar detecção USB:**
```bash
lsusb   # Samsung aparece como ID 04e8:xxxx
```

**Testar impressão:**
```bash
system-config-printer
# Botão direito na impressora → Imprimir página de teste
```

---

## Impressoras de rede Wi-Fi (uso corporativo)

Quando conectado à rede da empresa, o CUPS escaneia e lista as impressoras
disponíveis automaticamente.

**Passos:**
1. Conecte ao Wi-Fi da empresa
2. Abra: `system-config-printer`
3. Clique em **Adicionar**
4. Selecione a impressora na lista e instale o driver sugerido

**Se a impressora não aparecer automaticamente**, solicite ao TI:
- Endereço IP da impressora, ou
- URI no formato `ipp://192.168.x.x/ipp/print`

---

## Comandos úteis

```bash
# Listar impressoras configuradas
lpstat -p

# Verificar status do CUPS
systemctl status cups

# Listar backends/dispositivos detectados
lpinfo -v
```
