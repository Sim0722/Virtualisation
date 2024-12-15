# TP2 : Routage, DHCP et DNS

## I- Routage

**Configuration de `router.tp2.efrei`**

Tout d'abord on configure l'interface réseau connectée au NAT pour récupérer une adresse IP via DHCP:

pour ce faire on défini un hostname avec la commande :

- sudo hostnamectl set-hostname router.tp2.efrei

Puis pour éviter que le firewall rejette les paquets IPs qui ne lui sont pas destinés on exécute la commande : 

- sudo firewall-cmd --add-masquerade

Enfin la commande pour que ce paramètre persiste :

- sudo firewall-cmd --add-maquerade --permanent 


Ensuite, on attribue une IP statique à la machine : 

- sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

- on rentre les infos suivantes :

```
DEVICE=enp0s8;
NAME=lan;

ONBOOT=yes;
BOOTPROTO=static;

IPADDR=10.2.1.254;
NETMASK=255.255.255.0;
```
Pour que la config s'applique :

- sudo nmcli con reload

- sudo nmcli con up lan

Enfin on fait un ip -a pour vérifier que tous nos paramètres sont corrects : 

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fd:79:7a brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.216/24 brd 192.168.122.255 scope global dynamic noprefixroute enp0s3
       valid_lft 2892sec preferred_lft 2892sec
    inet6 fe80::a00:27ff:fefd:797a/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:8f:24:77 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.254/24 brd 10.2.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe8f:2477/64 scope link 
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d4:75:23 brd ff:ff:ff:ff:ff:ff
    inet 192.168.57.10/24 brd 192.168.57.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed4:7523/64 scope link 
       valid_lft forever preferred_lft forever
```

Note : J'ai ajouté une troisième carte réseau, je lui ai attribué une IP statique afin que je puisse me connecter en ssh à la machine.

**Configuration de `node1.tp2.efrei`**

Configurons l'IP statique d'un VPCS :

- ip 10.2.1.1/24

Now, peut-on joindre le routeur ?

- ping 10.2.1.254

84 bytes from 10.2.1.254 icmp_seq=1 ttl=64 time=0.692 ms
84 bytes from 10.2.1.254 icmp_seq=2 ttl=64 time=1.634 ms
84 bytes from 10.2.1.254 icmp_seq=3 ttl=64 time=1.788 ms
84 bytes from 10.2.1.254 icmp_seq=4 ttl=64 time=1.604 ms
84 bytes from 10.2.1.254 icmp_seq=5 ttl=64 time=3.580 ms

Le ping fonctionne on peut joindre le routeur.

On veut ajouter une passerelle par défaut, on tape la commande :

- ip 10.2.1.1 255.255.255.0 10.2.1.254

On veut savoir si on a accès à internet :

- ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=8.668 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=11.578 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=11.246 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=59 time=11.469 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=59 time=11.016 ms

Let's go ça fonctionne !! 

Enfin on veut prouver que les paquets passent bien par le routeur pour cela on utilise la commande :

- trace 1.1.1.1

trace to 1.1.1.1, 8 hops max, press Ctrl+C to stop
 1   10.2.1.254   3.945 ms  1.846 ms  1.603 ms
 2   192.168.122.1   3.310 ms  3.146 ms  3.897 ms
 3   10.0.3.2   4.254 ms  3.536 ms  3.554 ms
 4     *  *  *
 5     *  *  *
 6   193.253.94.58   10.977 ms  9.061 ms  20.265 ms
 7     *  *  *
 8     *  *  *

On voit bien que la première adresse IP est celle du routeur, donc les paquets passent par le routeur.

**Afficher la CAM Table du switch**

La CAM table contient les infos de node1 branché sur le port Et 0/0

- show mac address-table

          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6800    DYNAMIC     Et0/1
   1    0800.278f.2477    DYNAMIC     Et2/0
Total Mac Addresses for this criterion: 2

## II- Server DHCP

**Install et conf du DHCP**

Tout d'abord on attribue une IP statique à la machine : 

- sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

- on rentre les infos suivantes :

```
DEVICE=enp0s3;
NAME=lan;

ONBOOT=yes;
BOOTPROTO=static;

IPADDR=10.2.1.253;
NETMASK=255.255.255.0;

```
Pour que la config s'applique :

- sudo nmcli con reload

- sudo nmcli con up lan

On configure le DHCP :

- sudo nano /etc/dhcp/dhcpd.conf

- on rentre les infos suivantes : 

```

authoritative;

subnet 10.1.1.0 netmask 255.255.255.0 {
        range dynamic-bootp 10.2.1.10 10.2.1.50;
        option broadcast-address 10.1.1.255;
        option routeurs 10.2.1.254;
}
```

- On ouvre le port car par défaut sur les machines rocky linux le firewall bloque toutes les requêtes :

- sudo firewall-cmd --add-service=dhcp

- sudo firewall-cmd --runtime-to-permanent

**Test du DHCP**

On fait en sorte que la machine récupère une IP dynamique :

- dhcp

DDORA IP 10.2.1.11/24 GW 10.2.1.254

On récupère bien une IP dans la range configurée et on voit que la passerelle est correcte. 

De plus, on peut joindre internet :

- ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=9.035 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=11.474 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=10.538 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=59 time=11.321 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=59 time=10.633 ms

**Bonus**

Ici, il suffit de reprendre la conf dhcp et de rajouter une ligne : 

```
option domain-name-servers     1.1.1.1;

authoritative;

subnet 10.1.1.0 netmask 255.255.255.0 {
        range dynamic-bootp 10.2.1.10 10.2.1.50;
        option broadcast-address 10.1.1.255;
        option routeurs 10.2.1.254;
}
```
On vérifie :

 show ip 

NAME        : VPCS[1]
IP/MASK     : 10.2.1.11/24
GATEWAY     : 10.2.1.254
DNS         : 1.1.1.1  
DHCP SERVER : 10.2.1.253
DHCP LEASE  : 42791, 43200/21600/37800
MAC         : 00:50:79:66:68:00
LPORT       : 20006
RHOST:PORT  : 127.0.0.1:20007
MTU         : 1500

On voit que le DNS est bien renseigné.

**Wireshark it !**

[DORA](capture/DORA.pcapng)

## III- ARP

### 1. Les tables ARP

**Affichez la table ARP de `router.tp2.efrei`**

Pour afficher la table ARP du routeur on exécute la commande :

- ip n s

10.2.1.11 dev enp0s8 lladdr 00:50:79:66:68:00 STALE 
10.2.1.253 dev enp0s8 lladdr 08:00:27:5e:90:c2 STALE 

**Capturez l'échange ARP avec Wireshark**

[ARP](capture/ARP.pcapng)

### 2. ARP Poisoning

**Envoyez une trame ARP arbitraire**

Depuis la kali on entre la commande :

- sudo arping -s 11:11:11:11:11:11 -S 10.2.1.254 10.2.1.11

résultat dans la table arp de node1 :

- show arp

11:11:11:11:11:11  10.2.1.254 expires in 101 seconds 

**Mettre en place un ARP MITM**

Pour faire un MiTM on tape la commande :

- sudo arpspoof -i eth0 -t 10.2.1.11 -r 10.2.1.254

ici -i : interface de la carte réseau du pc attaquant
ensuite -t : t pour target et on rentre l'IP de la machine visée
enfin -r : pour empoisonner à la fois la victime et la machine à laquelle elle se connecte et ainsi capturer le traffic dans les deux sens

**Capture Wireshark `arp_mitm.pcap`**

[ARP spoofing](capture/ARP_spoof.pcapng)

**Réaliser une attaque avec scapy**
