# Install Wireguard

```sh
# Install packages
apt install wireguard dkms raspberrypi-kernel-headers wireguard-dkms wireguard-tools

# Reboot after install raspberrypi-kernel-headers

# Generate keys
cd /etc/wireguard
umask 077

# Server keys
wg genkey > server_private.key
wg pubkey > server_public.key < server_private.key

# Client keys
wg genkey > client1_private.key 
wg pubkey > client1_public.key < client1_private.key
```

/etc/wireguard/wg0.conf

```conf
[Interface]
# Plage d'adresse privée allouée pour le VPN
Address = 10.10.0.0/24 
# Port d'écoute
ListenPort = 51820
# IP du DNS ici Quad9
DNS = 9.9.9.9
# Clé privée du serveur
PrivateKey = # PrivateKey for server
# Règles de routage en fonction de l'interface eth0
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
[Peer]
# Client 1
PublicKey = # PublicKey for client
# IP pour le client 1
AllowedIPs = 10.10.0.1/32
PersistentkeepAlive = 60
```

/etc/wireguard/wg0-client1.conf

```conf
[Interface]
# Client
# IP 
Address = 10.10.0.1/32
# Clé privée
PrivateKey = # PrivateKey for client

[Peer]
# Clé public du serveur
PublicKey = # PublicKey for server

# IP publique pour atteindre le VPN
Endpoint = 
# Permettre à tout le trafic d'être routé vers le VPN
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

/etc/sysctl.conf

```conf
net.ipv4.ip_forward=1
```

```sh
# Recharger paramètres du noyau
sysctl -p /etc/sysctl.conf

# Activer l'interface
wg-quick up wg0

# Vérifier
ip a

# Status de la connexion
wg

# Activer l'interface après un redémarrage
systemctl enable wg-quick@wg0
```