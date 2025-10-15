# Passo a Passo Completo – Proxy, Bloqueio de Sites e SSH

Este documento descreve o passo a passo detalhado para configurar o ambiente solicitado, desde o Windows até o Linux do grupo e do ISP, incluindo proxy, bloqueio de sites, servidor web e SSH.

---

## 1️⃣ Windows

### 1.1 Configurar proxy no Firefox

1. Abra o Firefox → Menu → Configurações → Geral → Configurações de Rede.
2. Selecione **Configuração manual de proxy**.
3. Informe:
   - HTTP Proxy: `200.10.0.41`
   - Porta: `3128`
   - Marque **Usar este proxy para todos os protocolos**
4. Clique em **OK**

### 1.2 Testar conexão

```cmd
ping 200.10.0.41
```
- Verifique se o Linux responde.

### 1.3 Testar SSH

```powershell
ssh usuario_grupo@200.10.0.41
```
- Inicialmente pode dar timeout se o SSH ainda não estiver configurado.

---

## 2️⃣ Linux do grupo

### 2.1 Atualizar sistema

```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Criar usuário e adicionar ao sudo

```bash
sudo adduser usuario_grupo
sudo usermod -aG sudo usuario_grupo
```

### 2.3 Configurar IP da LAN do grupo

```bash
sudo ip addr add 200.10.0.41/29 dev enp0s31f6
sudo ip link set enp0s31f6 up
```

Persistente via Netplan:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s31f6:
      dhcp4: no
      addresses: [200.10.0.41/29]
```
```bash
sudo netplan apply
```

### 2.4 Instalar pacotes

```bash
sudo apt install apache2 squid openssh-server -y
```

### 2.5 Criar página de bloqueio

```bash
sudo nano /var/www/html/blocked.html
```
Conteúdo:
```html
<html>
<head><title>Site Bloqueado</title></head>
<body>
<h1>Atenção!</h1>
<p>O site solicitado está bloqueado pelo proxy do grupo.</p>
</body>
</html>
```

### 2.6 Configurar Squid

```bash
sudo nano /etc/squid/squid.conf
```
Adicionar:
```conf
http_port 3128
acl blocked_sites dstdomain .bet365.bet.br
http_access deny blocked_sites
deny_info http://200.10.0.41/blocked.html blocked_sites
acl localnet src 200.10.0.0/24
http_access allow localnet
http_access deny all
```

### 2.7 Reiniciar Squid

```bash
sudo systemctl restart squid
```


### 2.7.6 CONFIGURAR O SSH DO LINUX 

-Instalar servidor SSH
```bash
sudo apt install openssh-server -y
```

-Verificar status do SSH
```bash
sudo systemctl status ssh


```
```bash
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo ufw allow OpenSSH
sudo ufw enable
```

-Teste do Windows ou Linux do grupo:
```bash
ssh usuario_grupo@200.10.0.41
```



### 2.8 Testar bloqueio no Windows
- Abra o Firefox configurado com proxy e acesse `http://bet365.bet.br`. Deve redirecionar para `blocked.html`.

### 2.9 Monitorar logs

```bash
sudo tail -f /var/log/squid/access.log
```
- Para filtrar bloqueios:
```bash
sudo grep blocked.html /var/log/squid/access.log
```
- Ctrl+C para sair.

### 2.10 Testar SSH

```powershell
ssh usuario_grupo@200.10.0.41
```
- Deve conectar.

---

## 3️⃣ Servidor Linux do ISP (Lab)

### 3.1 Configurar SSH

```bash
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

### 3.2 Criar usuário e adicionar ao sudo

```bash
sudo adduser usuario_grupo
sudo usermod -aG sudo usuario_grupo
```

### 3.3 Instalar e configurar proxy transparente

```bash
sudo apt install squid -y
```
Configuração mínima para acesso total:
```conf
http_port 3128 transparent
acl localnet src 0.0.0.0/0
http_access allow localnet
http_access deny all
```
```bash
sudo systemctl restart squid
```

### 3.4 Configurar subinterface com IP WAN do grupo
```bash
sudo ip addr add 192.168.0.20/24 dev enp0s31f6
sudo ip link set enp0s31f6 up
```

### 3.5 Rotear range de IP do grupo
```bash
sudo ip route add 200.10.0.0/24 via 192.168.0.20
```

### 3.6 Testar acesso entre grupos
- Todos os grupos devem conseguir acessar a página dos outros grupos usando proxy ou navegador apontando para a LAN correta.

---




## 4️⃣ Entrega e Observações
- Entrega: até 21/10
- Disponibilidade para execução no laboratório 316: 08/10 e 15/10
- A partir de 22/10 iniciamos com RIP

> Este passo a passo cobre a configuração inicial de proxy, bloqueio de sites, SSH, criação de usuários e IPs de LAN/WAN, pronto para ser seguido em laboratório.

