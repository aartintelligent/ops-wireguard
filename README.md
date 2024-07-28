# WIREGUARD NETWORK

## SERVER

```shell
sudo nano /etc/wireguard/wg0.conf
```

```text
### vpn-server

[Interface]
Address = 10.200.0.1/16
ListenPort = 51194
PrivateKey = CB6ZXAQAga4CkkJavjqNBbr40dvDdrziSV2rfNm9WFo=
SaveConfig = false
PostUp = /etc/wireguard/post_up.sh
PostDown = /etc/wireguard/post_down.sh

### vpn-peer-01

[Peer]
PublicKey = flau4eoBf6M+kC3VKP673mj1o+ZscTKpAKhaD3oJO3o=
AllowedIPs = 10.200.0.10

### vpn-peer-02

[Peer]
PublicKey = yZQRPLyHmpfVwTSrsOVaR+E/e3UUoxkejE/V0K8gzmU=
AllowedIPs = 10.200.0.20
```

```shell
sudo nano /etc/wireguard/post_up.sh
```

```text
#!/bin/bash
set -e

WIREGUARD_INTERFACE="wg0"

WIREGUARD_LAN="10.200.0.0/16"

MASQUERADE_INTERFACE="eth1"

CHAIN_NAME="WIREGUARD_$WIREGUARD_INTERFACE"

sysctl -w net.ipv4.ip_forward=1

iptables -P FORWARD DROP

iptables -t nat -I POSTROUTING -o $MASQUERADE_INTERFACE -j MASQUERADE -s $WIREGUARD_LAN

iptables -N $CHAIN_NAME 2>/dev/null || true

iptables -A FORWARD -j $CHAIN_NAME

iptables -A $CHAIN_NAME -o $WIREGUARD_INTERFACE -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

iptables -A $CHAIN_NAME -s $WIREGUARD_LAN -i $WIREGUARD_INTERFACE -j ACCEPT

iptables -A $CHAIN_NAME -i $WIREGUARD_INTERFACE -j DROP

iptables -A $CHAIN_NAME -j RETURN

iptables -A INPUT -p icmp -j ACCEPT

iptables -A FORWARD -i $WIREGUARD_INTERFACE -o $WIREGUARD_INTERFACE -j ACCEPT

exit 0
```

```shell
sudo chmod +x /etc/wireguard/post_up.sh
```

```shell
sudo nano /etc/wireguard/post_down.sh
```

```text
#!/bin/bash
set -e

WIREGUARD_INTERFACE="wg0"

WIREGUARD_LAN="10.200.0.0/16"

MASQUERADE_INTERFACE="eth1"

CHAIN_NAME="WIREGUARD_$WIREGUARD_INTERFACE"

iptables -t nat -D POSTROUTING -o $MASQUERADE_INTERFACE -j MASQUERADE -s $WIREGUARD_LAN

iptables -D FORWARD -j $CHAIN_NAME

iptables -F $CHAIN_NAME

iptables -X $CHAIN_NAME

iptables -D FORWARD -i $WIREGUARD_INTERFACE -o $WIREGUARD_INTERFACE -j ACCEPT

sysctl -w net.ipv4.ip_forward=0

exit 0
```

```shell
sudo chmod +x /etc/wireguard/post_down.sh
```

```shell
sudo systemctl enable wg-quick@wg0
 
sudo systemctl start wg-quick@wg0
```

## PEERS

### PEER 1

```shell
sudo nano /etc/wireguard/wg0.conf
```

```text
### vpn-peer-01

[Interface]
Address = 10.200.0.10/32
PrivateKey = aMz7xx4aNS349Ksqa2Y60w7gtXeEFFFusioLMUdHYEg=
ListenPort = 51194

[Peer]
PublicKey = vgR1t+KAyHkg68cPyJg2HxDSFucUmrS4apqD6P6x9lg=
AllowedIPs = 10.200.0.0/16
Endpoint = 192.168.56.10:51194
PersistentKeepalive = 15
```

```text
sudo systemctl enable wg-quick@wg0
 
sudo systemctl start wg-quick@wg0

ping -c 4 10.200.0.1
```

### PEER 2

```shell
sudo nano /etc/wireguard/wg0.conf
```

```text
### vpn-peer-02

[Interface]
Address = 10.200.0.20/32
PrivateKey = YGKpOqVDr7ExB2UqFk2Gp57S4cwSPWmV50PfQ9Ib3G4=
ListenPort = 51194

[Peer]
PublicKey = vgR1t+KAyHkg68cPyJg2HxDSFucUmrS4apqD6P6x9lg=
AllowedIPs = 10.200.0.0/16
Endpoint = 192.168.56.10:51194
PersistentKeepalive = 15
```

```shell
sudo systemctl enable wg-quick@wg0
 
sudo systemctl start wg-quick@wg0

ping -c 4 10.200.0.1
```
