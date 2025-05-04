# vpn-local
Repositório destinado a um projeto para criar uma VPN local (Windows)

### Passo 1: **Instalar o OpenVPN no PC (Servidor)**

1. **Baixe o OpenVPN**:

   * Vá para o site oficial do OpenVPN: [OpenVPN Community Downloads](https://openvpn.net/community-downloads/).
   * Baixe o instalador compatível com seu sistema (Windows).

2. **Instale o OpenVPN**:

   * Execute o instalador e siga as instruções para instalar o OpenVPN.
   * Durante a instalação, **marque a opção de instalar o OpenVPN Service** e **os drivers TAP** (caso não encontre esses nomes, pode estar chamado como EasyRSA).
   * Após a instalação, opcionalmente reinicie o computador para garantir que o serviço seja iniciado corretamente.

---

### Passo 2: **Configurar o Servidor OpenVPN**

1. **Configuração do EasyRSA**:
   O OpenVPN utiliza o **EasyRSA** para criar certificados SSL que são usados para autenticação. O EasyRSA já vem incluso no OpenVPN.

   * Abra o **Prompt de Comando** (cmd) ou PowerShell como administrador.
   * Navegue até o diretório onde o OpenVPN foi instalado. Normalmente, ele fica em algo como:

     ```
     cd C:\Program Files\OpenVPN\easy-rsa
     ```

2. **Gerar Certificados e Chaves**:

   * Dentro da pasta `easy-rsa`, execute os seguintes comandos para criar a estrutura de certificados:

     * Primeiro, inicialize o EasyRSA:
    
       ```
       ./EasyRSA-Start.bat
       ```
     * Após, inicialize o PKI:

       ```
       easyrsa init-pki
       ```
     * Em seguida, crie a chave mestre para o servidor (deixe o nome default: server):

       ```
       easyrsa gen-req server nopass
       ```

       Isso criará a chave do servidor.

     * Agora crie um certificado CA e defina uma senha (exemplo: cava):
    
       ```
       easyrsa build-ca
       ```
       
     * Agora, assine a chave do servidor:

       ```
       easyrsa sign-req server server
       ```
     * Crie a chave de cliente (este será o arquivo que você enviará para seu celular mais tarde):

       ```
       easyrsa gen-req cliente nopass
       easyrsa sign-req client cliente
       ```
     * Finalmente, crie a chave Diffie-Hellman (necessária para criptografia):

       ```
       easyrsa gen-dh
       ```

3. **Configurar o arquivo de configuração do servidor**:
   O OpenVPN usa arquivos de configuração `.ovpn`. Vamos criar o arquivo de configuração do servidor.

   * Saia do `easyrsa` e continue usando o terminal.
   * Vá até o diretório `C:\Users\<Seu Usuário>\OpenVPN\config`.
   * Crie um arquivo chamado `server.ovpn` (touch server.ovpn).
   * Insira o seguinte conteúdo básico no arquivo (nano ./server.ovpn), para facilitar e não precisar encaminhar muitos arquivos ca.crt, dh.pem, server.key, server.crt, vamos adicionar tudo no mesmo arquivo:

      ```
      #################################################
      # Sample OpenVPN 2.6 config file for            #
      # multi-client server.                          #
      #                                               #
      # This file is for the server side              #
      # of a many-clients <-> one-server              #
      # OpenVPN configuration.                        #
      #                                               #
      # OpenVPN also supports                         #
      # single-machine <-> single-machine             #
      # configurations (See the Examples page         #
      # on the web site for more info).               #
      #                                               #
      # This config should work on Windows            #
      # or Linux/BSD systems.  Remember on            #
      # Windows to quote pathnames and use            #
      # double backslashes, e.g.:                     #
      # "C:\\Program Files\\OpenVPN\\config\\foo.key" #
      #                                               #
      # Comments are preceded with '#' or ';'         #
      #################################################
      
      # Which local IP address should OpenVPN
      # listen on? (optional)
      ;local a.b.c.d
      
      # Which TCP/UDP port should OpenVPN listen on?
      # If you want to run multiple OpenVPN instances
      # on the same machine, use a different port
      # number for each one.  You will need to
      # open up this port on your firewall.
      port 1194
      
      # TCP or UDP server?
      ;proto tcp
      proto udp
      
      # "dev tun" will create a routed IP tunnel,
      # "dev tap" will create an ethernet tunnel.
      # Use "dev tap0" if you are ethernet bridging
      # and have precreated a tap0 virtual interface
      # and bridged it with your ethernet interface.
      # If you want to control access policies
      # over the VPN, you must create firewall
      # rules for the TUN/TAP interface.
      # On non-Windows systems, you can give
      # an explicit unit number, such as tun0.
      # On Windows, use "dev-node" for this.
      # On most systems, the VPN will not function
      # unless you partially or fully disable/open
      # the firewall for the TUN/TAP interface.
      ;dev tap
      dev tun
      
      # Windows needs the TAP-Win32 adapter name
      # from the Network Connections panel if you
      # have more than one.
      # You may need to selectively disable the
      # Windows firewall for the TAP adapter.
      # Non-Windows systems usually don't need this.
      ;dev-node MyTap
      
      # SSL/TLS root certificate (ca), certificate
      # (cert), and private key (key).  Each client
      # and the server must have their own cert and
      # key file.  The server and all clients will
      # use the same ca file.
      #
      # See the "easy-rsa" project at
      # https://github.com/OpenVPN/easy-rsa
      # for generating RSA certificates
      # and private keys.  Remember to use
      # a unique Common Name for the server
      # and each of the client certificates.
      #
      # Any X509 key management system can be used.
      # OpenVPN can also use a PKCS #12 formatted key file
      # (see "pkcs12" directive in man page).
      #
      # If you do not want to maintain a CA
      # and have a small number of clients
      # you can also use self-signed certificates
      # and use the peer-fingerprint option.
      # See openvpn-examples man page for a
      # configuration example.
      ca ca.crt
      cert server.crt
      key server.key  # This file should be kept secret
      
      # Diffie hellman parameters.
      # Generate your own with:
      #   openssl dhparam -out dh2048.pem 2048
      dh dh.pem
      
      # Allow to connect to really old OpenVPN versions
      # without AEAD support (OpenVPN 2.3.x or older)
      # This adds AES-256-CBC as fallback cipher and
      # keeps the modern ciphers as well.
      data-ciphers AES-256-GCM:AES-128-GCM:?CHACHA20-POLY1305:AES-256-CBC
      
      # Network topology
      # Should be subnet (addressing via IP)
      # unless Windows clients v2.0.9 and lower have to
      # be supported (then net30, i.e. a /30 per client)
      # Defaults to net30 (not recommended)
      topology subnet
      
      # Configure server mode and supply a VPN subnet
      # for OpenVPN to draw client addresses from.
      # The server will take 10.8.0.1 for itself,
      # the rest will be made available to clients.
      # Each client will be able to reach the server
      # on 10.8.0.1. Comment this line out if you are
      # ethernet bridging. See the man page for more info.
      server 10.8.0.0 255.255.255.0
      
      # Maintain a record of client <-> virtual IP address
      # associations in this file.  If OpenVPN goes down or
      # is restarted, reconnecting clients can be assigned
      # the same virtual IP address from the pool that was
      # previously assigned.
      ifconfig-pool-persist ipp.txt
      
      # Configure server mode for ethernet bridging.
      # You must first use your OS's bridging capability
      # to bridge the TAP interface with the ethernet
      # NIC interface.  Then you must manually set the
      # IP/netmask on the bridge interface, here we
      # assume 10.8.0.4/255.255.255.0.  Finally we
      # must set aside an IP range in this subnet
      # (start=10.8.0.50 end=10.8.0.100) to allocate
      # to connecting clients.  Leave this line commented
      # out unless you are ethernet bridging.
      ;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100
      
      # Configure server mode for ethernet bridging
      # using a DHCP-proxy, where clients talk
      # to the OpenVPN server-side DHCP server
      # to receive their IP address allocation
      # and DNS server addresses.  You must first use
      # your OS's bridging capability to bridge the TAP
      # interface with the ethernet NIC interface.
      # Note: this mode only works on clients (such as
      # Windows), where the client-side TAP adapter is
      # bound to a DHCP client.
      ;server-bridge
      
      # Push routes to the client to allow it
      # to reach other private subnets behind
      # the server.  Remember that these
      # private subnets will also need
      # to know to route the OpenVPN client
      # address pool (10.8.0.0/255.255.255.0)
      # back to the OpenVPN server.
      ;push "route 192.168.10.0 255.255.255.0"
      ;push "route 192.168.20.0 255.255.255.0"
      
      # To assign specific IP addresses to specific
      # clients or if a connecting client has a private
      # subnet behind it that should also have VPN access,
      # use the subdirectory "ccd" for client-specific
      # configuration files (see man page for more info).
      
      # EXAMPLE: Suppose the client
      # having the certificate common name "Thelonious"
      # also has a small subnet behind his connecting
      # machine, such as 192.168.40.128/255.255.255.248.
      # First, uncomment out these lines:
      ;client-config-dir ccd
      ;route 192.168.40.128 255.255.255.248
      # Then create a file ccd/Thelonious with this line:
      #   iroute 192.168.40.128 255.255.255.248
      # This will allow Thelonious' private subnet to
      # access the VPN.  This example will only work
      # if you are routing, not bridging, i.e. you are
      # using "dev tun" and "server" directives.
      
      # EXAMPLE: Suppose you want to give
      # Thelonious a fixed VPN IP address of 10.9.0.1.
      # First uncomment out these lines:
      ;client-config-dir ccd
      ;route 10.9.0.0 255.255.255.252
      # Then add this line to ccd/Thelonious:
      #   ifconfig-push 10.9.0.1 10.9.0.2
      
      # Suppose that you want to enable different
      # firewall access policies for different groups
      # of clients.  There are two methods:
      # (1) Run multiple OpenVPN daemons, one for each
      #     group, and firewall the TUN/TAP interface
      #     for each group/daemon appropriately.
      # (2) (Advanced) Create a script to dynamically
      #     modify the firewall in response to access
      #     from different clients.  See man
      #     page for more info on learn-address script.
      ;learn-address ./script
      
      # If enabled, this directive will configure
      # all clients to redirect their default
      # network gateway through the VPN, causing
      # all IP traffic such as web browsing and
      # DNS lookups to go through the VPN
      # (The OpenVPN server machine may need to NAT
      # or bridge the TUN/TAP interface to the internet
      # in order for this to work properly).
      push "redirect-gateway def1 bypass-dhcp"
      
      # Certain Windows-specific network settings
      # can be pushed to clients, such as DNS
      # or WINS server addresses.  CAVEAT:
      # http://openvpn.net/faq.html#dhcpcaveats
      # The addresses below refer to the public
      # DNS servers provided by opendns.com.
      push "dhcp-option DNS 208.67.222.222"
      push "dhcp-option DNS 208.67.220.220"
      
      # Uncomment this directive to allow different
      # clients to be able to "see" each other.
      # By default, clients will only see the server.
      # To force clients to only see the server, you
      # will also need to appropriately firewall the
      # server's TUN/TAP interface.
      ;client-to-client
      
      # Uncomment this directive if multiple clients
      # might connect with the same certificate/key
      # files or common names.  This is recommended
      # only for testing purposes.  For production use,
      # each client should have its own certificate/key
      # pair.
      #
      # IF YOU HAVE NOT GENERATED INDIVIDUAL
      # CERTIFICATE/KEY PAIRS FOR EACH CLIENT,
      # EACH HAVING ITS OWN UNIQUE "COMMON NAME",
      # UNCOMMENT THIS LINE.
      ;duplicate-cn
      
      # The keepalive directive causes ping-like
      # messages to be sent back and forth over
      # the link so that each side knows when
      # the other side has gone down.
      # Ping every 10 seconds, assume that remote
      # peer is down if no ping received during
      # a 120 second time period.
      keepalive 10 120
      
      # For extra security beyond that provided
      # by SSL/TLS, create an "HMAC firewall"
      # to help block DoS attacks and UDP port flooding.
      #
      # Generate with:
      #   openvpn --genkey tls-auth ta.key
      #
      # The server and each client must have
      # a copy of this key.
      # The second parameter should be '0'
      # on the server and '1' on the clients.
      ;tls-auth ta.key 0 # This file is secret
      
      # The maximum number of concurrently connected
      # clients we want to allow.
      ;max-clients 100
      
      # It's a good idea to reduce the OpenVPN
      # daemon's privileges after initialization.
      #
      # You can uncomment this on non-Windows
      # systems after creating a dedicated user.
      ;user openvpn
      ;group openvpn
      
      # The persist options will try to avoid
      # accessing certain resources on restart
      # that may no longer be accessible because
      # of the privilege downgrade.
      persist-key
      persist-tun
      
      # Output a short status file showing
      # current connections, truncated
      # and rewritten every minute.
      status openvpn-status.log
      
      # By default, log messages will go to the syslog (or
      # on Windows, if running as a service, they will go to
      # the "\Program Files\OpenVPN\log" directory).
      # Use log or log-append to override this default.
      # "log" will truncate the log file on OpenVPN startup,
      # while "log-append" will append to it.  Use one
      # or the other (but not both).
      ;log         openvpn.log
      ;log-append  openvpn.log
      
      # Set the appropriate level of log
      # file verbosity.
      #
      # 0 is silent, except for fatal errors
      # 4 is reasonable for general usage
      # 5 and 6 can help to debug connection problems
      # 9 is extremely verbose
      verb 3
      
      # Silence repeating messages.  At most 20
      # sequential messages of the same message
      # category will be output to the log.
      ;mute 20
      
      # Notify the client that when the server restarts so it
      # can automatically reconnect.
      explicit-exit-notify 1
      
      
      # Configurações de roteamento
      #push "redirect-gateway def1"
      #push "dhcp-option DNS 8.8.8.8"
      #push "dhcp-option DNS 8.8.4.4"
      
      # Compressão de dados usando o algoritmo LZO (pode melhorar a performance em conexões lentas).
      # comp-lzo
      ```

   Esse arquivo de configuração basicamente diz ao servidor OpenVPN como se conectar, quais portas e protocolos usar e onde encontrar os certificados.

   **Nota:** É necessário copiar para esta pasta os arquivos, `ca.crt`, `dh.pem`, `server.crt` e `server.key`, que estão presentes em `C:\Program Files\OpenVPN` (ou onde o OpenVPN foi instalado).

---

### Passo 3: **Redirecionar as Portas no Roteador**

1. **Acesse o Roteador**:

   * Abra o navegador e digite o IP do seu roteador. No Deco é `192.168.68.1`, mas normalmente, ele é `192.168.0.1` ou `192.168.1.1`.
   * Faça login com seu nome de usuário e senha (se você não souber, pode verificar no manual do roteador ou no adesivo do próprio roteador), geralmente uma senha sem caracteres especiais e apenas minúsculas.

2. **Redirecione a Porta 1194**:

   * Encontre a seção de **Encaminhamento de Portas** (Port Forwarding) em alguns casos pode estar escrito **Encaminhamento de NAT**.
   * Crie uma nova regra de encaminhamento para a porta **1194 UDP** (essa é a porta padrão do OpenVPN).

     * **Porta Externa/Interna**: 1194
     * **Protocolo**: UDP
     * **Endereço IP Interno**: O IP do seu PC onde o OpenVPN está instalado.
   * Salve as configurações.

---

### Passo 4: **Iniciar o servidor**

- Usar o OpenVPN GUI, você pode iniciá-lo a partir da barra de tarefas, conectar.

Clique no ícone do OpenVPN na bandeja do sistema (ao lado do relógio).

Selecione Conectar no perfil de conexão configurado.

### Passo 5: **Gerar o arquivo de configuração para o cliente**

```
##############################################
# Sample client-side OpenVPN 2.6 config file #
# for connecting to multi-client server.     #
#                                            #
# This configuration can be used by multiple #
# clients, however each client should have   #
# its own cert and key files.                #
#                                            #
# On Windows, you might want to rename this  #
# file so it has a .ovpn extension           #
##############################################

# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client

# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun

# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel
# if you have more than one.  On XP SP2,
# you may need to disable the firewall
# for the TAP adapter.
;dev-node MyTap

# Are we connecting to a TCP or
# UDP server?  Use the same setting as
# on the server.
;proto tcp
proto udp

# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote 187.45.117.5 1194
;remote my-server-2 1194

# Choose a random host from the remote
# list for load-balancing.  Otherwise
# try hosts in the order specified.
;remote-random

# Keep trying indefinitely to resolve the
# host name of the OpenVPN server.  Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite

# Most clients don't need to bind to
# a specific local port number.
nobind

# Downgrade privileges after initialization (non-Windows only)
;user openvpn
;group openvpn

# Try to preserve some state across restarts.
persist-key
persist-tun


# If you are connecting through an
# HTTP proxy to reach the actual OpenVPN
# server, put the proxy server/IP and
# port number here.  See the man page
# if your proxy server requires
# authentication.
;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]

# Wireless networks often produce a lot
# of duplicate packets.  Set this flag
# to silence duplicate packet warnings.
;mute-replay-warnings

# SSL/TLS parms.
# See the server config file for more
# description.  It's best to use
# a separate .crt/.key file pair
# for each client.  A single ca
# file can be used for all clients.
# ca ca.crt
# cert client.crt
# key client.key

# Verify server certificate by checking that the
# certificate has the correct key usage set.
# This is an important precaution to protect against
# a potential attack discussed here:
#  http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the keyUsage set to
#   digitalSignature, keyEncipherment
# and the extendedKeyUsage to
#   serverAuth
# EasyRSA can do this for you.
remote-cert-tls server

# Allow to connect to really old OpenVPN versions
# without AEAD support (OpenVPN 2.3.x or older)
# This adds AES-256-CBC as fallback cipher and
# keeps the modern ciphers as well.
data-ciphers AES-256-GCM:AES-128-GCM:?CHACHA20-POLY1305:AES-256-CBC

# If a tls-auth key is used on the server
# then every client must also have the key.
;tls-auth ta.key 1

# Set log file verbosity.
verb 3

# Silence repeating messages
;mute 20

<ca>
-----BEGIN CERTIFICATE-----
[conteúdo do arquivo ca.crt aqui]
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
[conteúdo do arquivo server.crt aqui]
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
[conteúdo do arquivo server.key aqui]
-----END PRIVATE KEY-----
</key>
```

**Nota:** Lembre-se de preencher com o conteúdo dos arquivos.

### Passo 4: **Instalar o OpenVPN no Celular**

1. **Baixe o App OpenVPN Connect**:

   * No **Android** ou **iOS**, baixe o app **OpenVPN Connect** na Google Play Store ou App Store.

2. **Transferir os Arquivos de Configuração para o Celular**:

   * No seu PC, você precisa enviar o arquivo de configuração gerado (o arquivo `client.ovpn` para o cliente) para o celular.
   * Você pode fazer isso de várias maneiras:

     * **E-mail**: Envie o arquivo para seu próprio e-mail e faça o download no celular.
     * **Transferência USB**: Se o celular estiver conectado ao PC via USB, basta copiar o arquivo para o dispositivo.
     * **Aplicativos de Compartilhamento**: Usar aplicativos como Google Drive ou Dropbox também funciona.

3. **Importar o Arquivo de Configuração no App**:

   * Abra o app **OpenVPN Connect** no celular.
   * Toque em **Importar** ou **+** e selecione o arquivo `.ovpn` que você transferiu.
   * O app pode pedir as credenciais que você configurou para o cliente (se tiver configurado um nome de usuário e senha).

---

### Passo 5: **Conectar-se à VPN**

1. **Conectar no OpenVPN**:

   * Após importar o arquivo `.ovpn` no app do celular, toque para conectar.
   * O OpenVPN tentará estabelecer uma conexão com o seu PC.
   * Caso a conexão seja bem-sucedida, você estará acessando a internet como se estivesse no mesmo local que o seu PC!

  ⚠️ Fui Até aqui, tentamos, mas sem sucesso (timeout).
---

### Passo 6 (Opcional): **Configuração do DDNS** (Para IP Dinâmico)

Se o seu IP público mudar frequentemente (como acontece com muitas conexões de internet residencial), pode ser interessante configurar um **DDNS** para associar um nome de domínio ao seu IP dinâmico.

* **No-IP** e **DuckDNS** são serviços gratuitos de DDNS. Você pode registrar um nome de domínio gratuito e configurar seu roteador ou computador para atualizar automaticamente o IP.
