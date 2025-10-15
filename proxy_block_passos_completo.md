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
<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>🚫 Site Bloqueado — Grupo Leonardo & Germano</title>
<style>
  :root{
    --bg1:#0f172a;
    --bg2:#071032;
    --accent:#ffb86b;
    --muted:#98a0b3;
    --card:#0b1220;
  }
  *{box-sizing:border-box}
  body{
    margin:0;
    font-family:Inter, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    background: radial-gradient(1200px 600px at 10% 10%, rgba(255,184,107,0.06), transparent),
                linear-gradient(180deg,var(--bg1),var(--bg2));
    color:#e6eef8;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
    min-height:100vh;
    display:flex;
    align-items:center;
    justify-content:center;
    padding:30px;
  }

  .card{
    width:100%;
    max-width:980px;
    background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border-radius:16px;
    padding:28px;
    box-shadow: 0 8px 30px rgba(2,6,23,0.6);
    display:grid;
    grid-template-columns: 220px 1fr;
    gap:24px;
    align-items:center;
  }

  .lock {
    width:180px;
    height:180px;
    margin:auto;
    display:flex;
    align-items:center;
    justify-content:center;
    position:relative;
  }

  .padlock {
    width:110px;
    height:86px;
    border-radius:10px;
    background:linear-gradient(180deg,#101826,#0b1220);
    border:3px solid rgba(255,255,255,0.06);
    box-shadow: inset 0 -6px 18px rgba(0,0,0,0.6);
    position:relative;
    display:flex;
    align-items:center;
    justify-content:center;
    font-size:36px;
    color:var(--accent);
  }
  .shackle{
    position:absolute;
    top:-34px;
    left:50%;
    transform:translateX(-50%);
    width:86px;
    height:86px;
    border-radius:50%;
    border:10px solid rgba(255,255,255,0.06);
    border-bottom-color:transparent;
    background:transparent;
    filter:drop-shadow(0 6px 12px rgba(0,0,0,0.5));
    animation: wobble 3s infinite ease-in-out;
  }
  @keyframes wobble{
    0%{ transform:translateX(-50%) rotate(-6deg) }
    50%{ transform:translateX(-50%) rotate(6deg) }
    100%{ transform:translateX(-50%) rotate(-6deg) }
  }

  .content h1{
    margin:0 0 8px 0;
    font-size:28px;
    letter-spacing:-0.2px;
  }
  .content p{
    margin:0 0 14px 0;
    color:var(--muted);
    line-height:1.45;
  }

  .big-msg{
    display:flex;
    gap:10px;
    align-items:center;
    margin-bottom:14px;
  }
  .emoji{
    font-size:34px;
    transform:translateY(2px);
  }
  .reason{
    background:linear-gradient(90deg, rgba(255,184,107,0.08), rgba(255,255,255,0.01));
    padding:10px 12px;
    border-radius:10px;
    color:#fefcf7;
    display:inline-block;
    font-weight:600;
    letter-spacing:0.2px;
  }

  .actions{
    display:flex;
    gap:10px;
    margin-top:8px;
    align-items:center;
  }
  .btn{
    background:transparent;
    border:1px solid rgba(255,255,255,0.06);
    color:var(--accent);
    padding:10px 14px;
    border-radius:10px;
    cursor:pointer;
    font-weight:600;
    backdrop-filter: blur(4px);
  }
  .btn.secondary{
    color:var(--muted);
    background:transparent;
  }
  .footer{
    margin-top:18px;
    font-size:13px;
    color:var(--muted);
    display:flex;
    justify-content:space-between;
    align-items:center;
  }

  .credit{
    display:flex;
    gap:8px;
    align-items:center;
    font-weight:700;
    color:#fff;
  }
  .contact{
    color:var(--muted);
  }

  /* small friendly animation for text */
  .wobble-text{
    display:inline-block;
    animation: floaty 4s infinite;
  }
  @keyframes floaty{
    0%{ transform: translateY(0) }
    50%{ transform: translateY(-6px) }
    100%{ transform: translateY(0) }
  }

  @media (max-width:720px){
    .card{ grid-template-columns: 1fr; padding:20px; gap:16px; }
    .lock{ width:140px; height:140px; }
  }
</style>
</head>
<body>
  <main class="card" role="alert" aria-live="polite">
    <div class="lock" aria-hidden="true">
      <div class="shackle" title="Cadeado do Proxy"></div>
      <div class="padlock" aria-hidden="true">🔒</div>
    </div>

    <div class="content">
      <div class="big-msg">
        <span class="emoji" aria-hidden="true">😄</span>
        <h1>Opa! A navegação deu uma guinada inesperada.</h1>
      </div>

      <p>
        O site que você tentou abrir foi gentilmente barrado pelo proxy do <span class="wobble-text">Grupo Leonardo &amp; Germano</span>.
        — aqui não entra (pelo menos por enquanto). Se for necessário acessar, fale com o time.
      </p>

      <div class="reason" role="note">Motivo: política de bloqueio do grupo / categoria restrita</div>

      <div class="actions" aria-hidden="false">
        <button class="btn" onclick="history.back()">⬅️ Voltar para a página anterior</button>
        <button class="btn secondary" onclick="report()" title="Pedir liberação">📨 Pedir liberação</button>
        <button class="btn secondary" onclick="showJoke()" title="Aliviar o clima">😂 Me conte uma piada</button>
      </div>

      <div class="footer">
        <div class="credit">Grupo Leonardo &amp; Germano</div>
        <div class="contact">Se precisar: fale com <strong>Leonardo</strong> ou <strong>Germano</strong></div>
      </div>
    </div>
  </main>

<script>
  function report(){
    // ação simples: abre o mailto (pode ser adaptado para enviar form via servidor do proxy)
    const subject = encodeURIComponent("Pedido de liberação de site bloqueado");
    const body = encodeURIComponent("Olá Leonardo/Germano,\n\nPeço revisão do bloqueio para o site: " + location.href + "\nMotivo/observações:\n");
    window.location.href = "mailto:leonardo@example.com,germano@example.com?subject=" + subject + "&body=" + body;
  }

  function showJoke(){
    const jokes = [
      "Por que o site foi ao médico? Porque estava com muitos 'bugs'! 🐞",
      "O administrador do proxy disse: 'Sem permissão, sem festa.' 🎉",
      "— A página bloqueada perguntou: 'Você me ama?' — Proxy: 'Só de forma controlada.' 😅"
    ];
    const j = jokes[Math.floor(Math.random()*jokes.length)];
    alert("Piada do proxy:\n\n" + j);
  }
</script>
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
acl localnet src 200.10.0.40/29
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

### 3.3 Configurar subinterface com IP WAN do grupo
```bash
sudo ip addr add 192.168.0.20/30 dev enp0s31f6
sudo ip link set enp0s31f6 up
```

### 3.4 Rotear range de IP do grupo
```bash
sudo ip route add 200.10.0.40/29 via 192.168.0.20
```

### 3.5 Testar acesso entre grupos
- Todos os grupos devem conseguir acessar a página dos outros grupos usando proxy ou navegador apontando para a LAN correta.

---




## 4️⃣ Entrega e Observações
- Entrega: até 21/10
- Disponibilidade para execução no laboratório 316: 08/10 e 15/10
- A partir de 22/10 iniciamos com RIP

> Este passo a passo cobre a configuração inicial de proxy, bloqueio de sites, SSH, criação de usuários e IPs de LAN/WAN, pronto para ser seguido em laboratório.

