# I- Setup réseau initial

## 3- Marche à suivre 

### B. On ajoute les routes vers les LAN

**ping d'un client du LAN1 vers un client du LAN 2 :** 

- ping 10.3.2.1            

    84 bytes from 10.3.2.1 icmp_seq=1 ttl=62 time=34.836 ms
    84 bytes from 10.3.2.1 icmp_seq=2 ttl=62 time=29.417 ms
    84 bytes from 10.3.2.1 icmp_seq=3 ttl=62 time=21.071 ms
    84 bytes from 10.3.2.1 icmp_seq=4 ttl=62 time=35.727 ms
    84 bytes from 10.3.2.1 icmp_seq=5 ttl=62 time=40.127 ms

**Capture Wireshark :** 

![ping_partie1](capture/ping_partie1.pcapng)

#### **Afficher les adresses MAC des routeurs :**

**routeur 1 :**

- show arp

    Protocol  Address          Age (min)  Hardware Addr   Type   Interface
    Internet  10.3.12.1               -   c401.058d.0010  ARPA   FastEthernet1/0
    Internet  10.3.12.2              39   c402.05ab.0000  ARPA   FastEthernet1/0
    Internet  10.3.1.1               15   0050.7966.6800  ARPA   FastEthernet0/1
    Internet  10.3.1.2               21   0050.7966.6801  ARPA   FastEthernet0/1
    Internet  192.168.122.1          20   5254.0023.04c2  ARPA   FastEthernet0/0
    Internet  192.168.122.145         -   c401.058d.0000  ARPA   FastEthernet0/0
    Internet  10.3.1.254              -   c401.058d.0001  ARPA   FastEthernet0/1

**routeur 2 :**

- sh arp

    Protocol  Address          Age (min)  Hardware Addr   Type   Interface
    Internet  10.3.12.1              42   c401.058d.0010  ARPA   FastEthernet0/0
    Internet  10.3.12.2               -   c402.05ab.0000  ARPA   FastEthernet0/0
    Internet  10.3.2.2               21   0050.7966.6803  ARPA   FastEthernet0/1
    Internet  10.3.2.1               18   0050.7966.6802  ARPA   FastEthernet0/1
    Internet  10.3.2.254              -   c402.05ab.0001  ARPA   FastEthernet0/1


### C. Accès Internet

**Prouvez que vous avez déjà un accès internet**

- ping 1.1.1.1

    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 88/110/128 ms

**Accès internet lan1**

- ping 1.1.1.1

    84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=50.060 ms
    84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=41.384 ms
    84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=41.068 ms
    84 bytes from 1.1.1.1 icmp_seq=4 ttl=59 time=41.335 ms
    84 bytes from 1.1.1.1 icmp_seq=5 ttl=59 time=49.374 ms

- trace 1.1.1.1

    trace to 1.1.1.1, 8 hops max, press Ctrl+C to stop
    1   10.3.1.254   19.269 ms  9.642 ms  9.708 ms
    2   192.168.122.1   18.971 ms  19.904 ms  19.741 ms
    3   10.0.3.2   19.565 ms  19.610 ms  20.023 ms
    4     *  *  *
    5     *  *  *
    6   140.91.250.2   50.664 ms  49.435 ms  49.296 ms
    7   140.204.102.68   60.124 ms  49.418 ms  50.354 ms
    8     *  *  *

**Accès internet lan2**

- ping 1.1.1.1

    84 bytes from 1.1.1.1 icmp_seq=1 ttl=57 time=69.558 ms
    84 bytes from 1.1.1.1 icmp_seq=2 ttl=57 time=57.252 ms
    84 bytes from 1.1.1.1 icmp_seq=3 ttl=57 time=57.626 ms
    84 bytes from 1.1.1.1 icmp_seq=4 ttl=57 time=58.590 ms
    84 bytes from 1.1.1.1 icmp_seq=5 ttl=57 time=57.562 ms

- trace 1.1.1.1

    trace to 1.1.1.1, 8 hops max, press Ctrl+C to stop
    1   10.3.2.254   5.043 ms  9.863 ms  9.985 ms
    2   10.3.12.1   29.562 ms  29.746 ms  30.037 ms
    3   192.168.122.1   30.772 ms  29.610 ms  30.052 ms
    4   10.0.3.2   30.148 ms  29.581 ms  29.963 ms
    5     *  *  *
    6     *  *  *
    7     *  *  *
    8   140.204.102.68   49.250 ms  60.033 ms  60.124 ms

# II- Routeur-on-a-stick

## 3. Marche à suivre

### A. VLANs

**Test de ping**

- ping 10.3.1.2

    84 bytes from 10.3.1.2 icmp_seq=1 ttl=64 time=1.935 ms
    84 bytes from 10.3.1.2 icmp_seq=2 ttl=64 time=2.613 ms
    84 bytes from 10.3.1.2 icmp_seq=3 ttl=64 time=3.454 ms
    84 bytes from 10.3.1.2 icmp_seq=4 ttl=64 time=3.952 ms
    84 bytes from 10.3.1.2 icmp_seq=5 ttl=64 time=3.904 ms

### B. Routeur

- sh ip int br

    Interface                  IP-Address      OK? Method Status                Protocol
    FastEthernet0/0            192.168.122.201 YES DHCP   up                    up      
    FastEthernet0/1            unassigned      YES unset  up                    up      
    FastEthernet0/1.10         10.3.1.254      YES manual up                    up      
    FastEthernet0/1.20         10.3.2.254      YES manual up                    up      
    FastEthernet0/1.30         10.3.3.254      YES manual up                    up      
    FastEthernet1/0            unassigned      YES unset  administratively down down    
    FastEthernet2/0            unassigned      YES unset  administratively down down    



- ping 10.3.1.254

    84 bytes from 10.3.1.254 icmp_seq=1 ttl=255 time=11.160 ms
    84 bytes from 10.3.1.254 icmp_seq=2 ttl=255 time=13.240 ms
    84 bytes from 10.3.1.254 icmp_seq=3 ttl=255 time=4.063 ms
    84 bytes from 10.3.1.254 icmp_seq=4 ttl=255 time=14.650 ms
    84 bytes from 10.3.1.254 icmp_seq=5 ttl=255 time=5.984 ms

#### **Test de ping : accès internet**
**Routeur**

- ping 1.1.1.1

    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 52/61/72 ms

**VPCS**

- PC1>ping 1.1.1.1

    84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=19.938 ms
    84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=19.786 ms
    84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=17.040 ms
    84 bytes from 1.1.1.1 icmp_seq=4 ttl=59 time=21.590 ms
    84 bytes from 1.1.1.1 icmp_seq=5 ttl=59 time=22.710 ms


- PC2> ping 1.1.1.1

    84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=20.863 ms
    84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=19.824 ms
    84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=20.104 ms
    84 bytes from 1.1.1.1 icmp_seq=4 ttl=59 time=20.309 ms
    84 bytes from 1.1.1.1 icmp_seq=5 ttl=59 time=15.901 ms

# III- Service dans le lan

## DHCP 

**Prouvez avec un VPCS**

- pc4.tp3.b2> ip dhcp

    DDORA IP 10.3.2.10/24 GW 10.3.2.254

- pc4.tp3.b2> sh ip

    NAME        : pc4.tp3.b2[1]
    IP/MASK     : 10.3.2.10/24
    GATEWAY     : 10.3.2.254
    DNS         : 1.1.1.1  
    DHCP SERVER : 10.3.2.253
    DHCP LEASE  : 585, 600/300/525
    MAC         : 00:50:79:66:68:03
    LPORT       : 20027
    RHOST:PORT  : 127.0.0.1:20028
    MTU         : 1500

## DNS 

**Tests de Résolution DNS**

- pc4.tp3.b2> sh ip

    NAME        : pc4.tp3.b2[1]
    IP/MASK     : 10.3.2.10/24
    GATEWAY     : 10.3.2.254
    DNS         : 10.3.3.1  
    DHCP SERVER : 10.3.2.253
    DHCP LEASE  : 471, 475/237/415
    MAC         : 00:50:79:66:68:03
    LPORT       : 20027
    RHOST:PORT  : 127.0.0.1:20028
    MTU         : 1500


**ping efrei.fr :**

- pc4.tp3.b2> ping efrei.fr
    
    efrei.fr resolved to 51.255.68.208

    efrei.fr icmp_seq=1 timeout
    84 bytes from 51.255.68.208 icmp_seq=2 ttl=61 time=51.403 ms
    84 bytes from 51.255.68.208 icmp_seq=3 ttl=61 time=43.233 ms

**ping dns.tp3.b2 :**

- pc4.tp3.b2> ping dns.tp3.b2
    
    dns.tp3.b2 resolved to 10.3.3.1

    84 bytes from 10.3.3.1 icmp_seq=1 ttl=63 time=20.147 ms
    84 bytes from 10.3.3.1 icmp_seq=2 ttl=63 time=16.475 ms
    84 bytes from 10.3.3.1 icmp_seq=3 ttl=63 time=18.781 ms
    84 bytes from 10.3.3.1 icmp_seq=4 ttl=63 time=17.942 ms

**Capture Wireshark**

![ping_part_dns](/capture)











