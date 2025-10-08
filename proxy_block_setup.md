# Relatório – Proxy e Bloqueio de Sites

Nessa atividade, nosso objetivo foi configurar um ambiente em que o Windows acessasse a internet passando pelo Linux, usando o Squid como proxy, e bloquear sites específicos, como `bet365.bet.br`.

## 1️⃣ Configuração do Windows

### 1.1 Configurar proxy no Firefox

1. Abra o Firefox → Menu → Configurações → Geral → Configurações de Rede.
2. Selecione **Configuração manual de proxy**.
3. Informe:
   - **HTTP Proxy:** `200.10.0.41`
   - **Porta:** `3128`
   - Marque **Usar este proxy para todos os protocolos**.
4. Clique em **OK**.

> Resultado: todo o tráfego do navegador passa pelo Linux.

### 1.2 Testar conexão com o Linux via ping

```cmd
ping 200.10.0.41
```
- Deve responder se o IP estiver configurado corretamente.

### 1.3 Testar SSH do Windows (opcional)

```powershell
ssh usuario@200.10.0.41
```
- Inicialmente pode dar `Connection timed out` se o Linux ainda não tiver IP da LAN configurado ou SSH não estiver ativo.

## 2️⃣ Configuração do Linux do grupo

### 2.1 Verificar interfaces e IP

```bash
ip addr show
```
- Observamos que o Linux só tinha o loopback (`127.0.0.1`) e o IP da rede normal (`10.104.16.8/26`).

### 2.2 Configurar IP da LAN do grupo (temporário)

```bash
sudo ip addr add 200.10.0.41/24 dev enp0s31f6
sudo ip link set enp0s31f6 up
```

> Para persistir após reinício, usar Netplan:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s31f6:
      dhcp4: no
      addresses: [200.10.0.41/24]
```
```bash
sudo netplan apply
```

### 2.3 Instalar pacotes necessários

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 squid openssh-server -y
```

### 2.4 Criar página de bloqueio

```bash
sudo nano /var/www/html/blocked.html
```
```html
<html>
<head><title>Site Bloqueado</title></head>
<body>
<h1>Atenção!</h1>
<p>O site solicitado está bloqueado pelo proxy do grupo.</p>
</body>
</html>
```
- Teste local: `firefox http://localhost/blocked.html`

### 2.5 Configurar Squid

```bash
sudo nano /etc/squid/squid.conf
```
Adicionar ou ajustar:

```conf
http_port 3128

acl blocked_sites dstdomain .bet365.bet.br
http_access deny blocked_sites
deny_info http://200.10.0.41/blocked.html blocked_sites

acl localnet src 200.10.0.0/24
http_access allow localnet
http_access deny all
```

### 2.6 Reiniciar Squid

```bash
sudo systemctl restart squid
```

### 2.7 Testar bloqueio no Windows

- Abra o Firefox configurado com proxy.
- Acesse `http://bet365.bet.br`.
- Deve ser redirecionado para `http://200.10.0.41/blocked.html`.

### 2.8 Ver logs do Squid

```bash
sudo tail -f /var/log/squid/access.log
```
- Mostra em tempo real as requisições do Windows.
- Para sair: **Ctrl + C**

Filtrar apenas os acessos bloqueados:

```bash
sudo grep blocked.html /var/log/squid/access.log
```

### 2.9 Criar usuário e adicionar ao sudo

```bash
sudo adduser usuario_grupo
sudo usermod -aG sudo usuario_grupo
```

### 2.🔟 Testar SSH

```powershell
ssh usuario_grupo@200.10.0.41
```
- Agora deve conectar, caso o serviço SSH esteja ativo:

```bash
sudo systemctl status ssh
sudo systemctl start ssh
```

### 2.11 (Opcional) Configurar subinterfaces e IP WAN
- Subinterface LAN já configurada (`200.10.0.41/24`).
- Interface física com IP WAN ainda será configurada futuramente.

## ✅ Resumo do funcionamento atual

1. O Windows acessa a internet **passando pelo Linux via Squid**.
2. Sites bloqueados (`bet365.bet.br`) **são redirecionados** para `blocked.html`.
3. Os **logs confirmam todas as requisições** do Windows.
4. O SSH está pronto para ser testado e usado assim que o IP da LAN estiver configurado.

> Até agora, o proxy e o bloqueio de sites estão funcionando corretamente, mesmo antes de configurar o servidor completo.

