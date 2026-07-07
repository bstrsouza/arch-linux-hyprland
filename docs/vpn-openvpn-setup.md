# Configuração de VPN — OpenVPN via NetworkManager

---

## Contexto

O NetworkManager suporta OpenVPN nativamente através do plugin `networkmanager-openvpn`, permitindo importar um arquivo `.ovpn` direto pelo `nm-connection-editor` — sem precisar rodar o `openvpn` manualmente via linha de comando. Usuário e senha ficam salvos no perfil da conexão, então a ativação/desativação do dia a dia é feita pelo menu da waybar (ver [`waybar-setup.md`](./waybar-setup.md#network)).

---

## Instalação

```bash
sudo pacman -S openvpn networkmanager-openvpn
```

Se o `nm-connection-editor` já estava aberto, reinicie-o (e o NetworkManager, se o plugin não aparecer na lista de tipos de VPN):

```bash
sudo systemctl restart NetworkManager
```

---

## Importando o arquivo `.ovpn`

1. Abra o editor: `nm-connection-editor` (ou clique direito no ícone de rede da waybar)
2. Clique no `+` (Adicionar) → **VPN** → **Import a saved VPN configuration...**
3. Selecione o arquivo `.ovpn` fornecido
4. Na aba **VPN**, preencha usuário e senha
5. Marque **Store the password for all users** para não precisar digitar a senha a cada conexão
6. Salve

**Verificar que a conexão foi criada:**
```bash
nmcli connection show
# Deve aparecer com TYPE vpn
```

> **Onde fica salva a senha:** ao marcar "Store the password for all users", o segredo é gravado direto em `/etc/NetworkManager/system-connections/<nome>.nmconnection`, arquivo legível apenas por root — por isso a VPN conecta sem prompt de senha.

---

## Conectar / Desconectar

**Dia a dia:** clique esquerdo no ícone de rede da waybar → `networkmanager_dmenu` lista a VPN junto com as redes Wi-Fi, com opção de conectar/desconectar (ver [`waybar-setup.md`](./waybar-setup.md#network)).

**Via terminal:**
```bash
nmcli connection up "NomeDaVPN"
nmcli connection down "NomeDaVPN"
```

**Conferir se está ativa:**
```bash
nmcli connection show --active
ip addr show tun0   # interface aparece só com a VPN conectada
```

---

## Comandos úteis

```bash
# Listar todas as conexões configuradas
nmcli connection show

# Detalhes de uma conexão específica
nmcli connection show "NomeDaVPN"

# Logs do NetworkManager em tempo real (útil para debugar falha de conexão)
journalctl -u NetworkManager -f
```

---

## Troubleshooting

**Tipo VPN não aparece no `nm-connection-editor`**
O plugin `networkmanager-openvpn` não foi detectado — reinicie o NetworkManager (`sudo systemctl restart NetworkManager`) ou a sessão gráfica.

**Erro "falha de autenticação" (`AUTH_FAILED`)**
Usuário/senha incorretos ou certificado expirado no arquivo `.ovpn`. Reimporte o arquivo se ele foi atualizado pelo provedor da VPN.

**Conecta mas sem acesso à rede interna**
Verifique se as rotas do túnel estão corretas — na aba **IPv4** do `nm-connection-editor`, confirme se "Use this connection only for resources on its network" está configurado de acordo com o que o provedor da VPN espera (split tunnel vs. túnel completo).
