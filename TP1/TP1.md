# TP1 : Remise dans le bain

## Most simplest LAN 

**Déterminer l'adresse MAC de vos deux machines**

| Machine | Adresse MAC |
| ------- | ------- |
| node1.tp1.efrei | 00:50:79:66:68:00 |
| node2.tp1.efrei | 00:50:79:66:68:01 |

**IP Statique**
Commande entrée pour attribuée l'IP statique sur les machine vpcs :
- ip 10.1.1.1/24
- ip 10.1.1.2/24

pour être sûr que l'ip a bien changé : show ip

**Ping**
node2.tp1.efrei = ping 10.1.1.2 depuis node1.tp1.efrei
node1.tp1.efrei = ping 10.1.1.1 depuis node2.tp1.efrei

**Wireshark**
[ping](capture/ping.pcapng)

**ARP**
la commande utilisée pour connaître la MAC de son correspondant avec un vcps est : arp

## Ajoutons un switch

**Déterminer l'adresse MAC de vos trois machines**

On réutilise `show` :

| Machine | Adresse MAC |
| ------- | ------- |
| node1.tp1.efrei | 00:50:79:66:68:00 |
| node2.tp1.efrei | 00:50:79:66:68:01 |
| node3.tp1.efrei | 00:50:79:66:68:02 |

**Définir une adresse IP statique**
 
Etant donné qu'on avait déjà attribué des IP statique pour node1 et node2 on doit juste faire la même chose pour node3
- ip 10.1.1.3/24

## Serveur DHCP

[DHCP](capture/ping_part2.pcapng)

### 1.Legit 

**Donner un accès internet à la machine `dhcp.tp1.efrei`**

On commence par installer le serveur DHCP à l'aide de la commande `sudo dnf install dhcp-server -y`

**Installer et configurer un serveur DHCP**

On attribue un IP statique à la machine : 

- sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
- on rentre les infos suivantes :
```
DEVICE=enp0s3;
NAME=lan;

ONBOOT=yes;
BOOTPROTO=static;

IPADDR=10.1.1.253;
NETMASK=255.255.255.0;
```
Pout que la config s'applique :
- sudo nmcli con reload
- sudo nmcli con up lan

On configure le DHCP :

- sudo nano /etc/dhcp/dhcpd.conf
- on rentre les infos suivantes : 
```
authoritative;

subnet 10.1.1.0 netmask 255.255.255.0 {
	range dynamic-bootp 10.1.1.10 10.1.1.50;
	option broadcast-address 10.1.1.255;
	option routeurs 10.1.1.1;
}
```

- On ouvre le port car par défaut sur les machines rocky linux le firewall bloque toutes les requêtes :

- sudo firewall-cmd --add-service=dhcp
- sudo firewall-cmd --runtime-to-permanent

**Récupérer des adresses IP automatiquement depuis les trois nodes**

| Machine | Adresse IP |
| ------- | ---------- |
| node1.tp1.efrei | 10.1.1.10 |
| node2.tp1.efrei | 10.1.1.11 |
| node3.tp1.efrei | 10.1.1.12 |

**Wireshark DORA**

[DHCP](capture/DORA.pcapng)

### 2.DHCP spoofing

**Configurer dnsmasq**

- sudo install dnsmasq -y
- sudo nano /etc/dnsmasq.conf
- on créer la conf :
``` 
port=0
dhcp-range=10.1.1.210,10.1.1.250,255.255.255.0,12h
dhcp-authoritative
interface=enp0s3
 
```
- sudo systemctl enable --now dnsmasq
- sudo firewall-cmd --add-service=dhcp
- sudo firewall-cmd --runtime-to-permanent 

**Tests**

On vérifie que l'on récupère des IP : 


| Machine | Adresse IP |
| ------- | ----------- |
| node1.tp1.efrei | 10.1.1.245 |
| node2.tp1.efrei | 10.1.1.246 |
| node3.tp1.efrei | 10.1.1.247 |


**Wireshark**

[DHCP](capture/DHCP_Spoof.pcapng)

