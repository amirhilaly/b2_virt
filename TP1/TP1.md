memo: https://gitlab.com/it4lik/b2-netvirt-2025/-/blob/main/docs/memo/rocky/index.md

## 3. Know your MAC

**🌞 Déterminer l'adresse MAC de vos deux machines**

On lance une p'tite commande ```show``` a sur nos machines:

PC1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
PC1    10.1.1.1/24          0.0.0.0           00:50:79:66:68:00  20002  127.0.0.1:20003
       fe80::250:79ff:fe66:6800/64


## 4. IP Setup
**🌞 Définir une IP statique sur les deux machines**

```
node1.tp1> ip 10.1.1.1 
node2.tp1> ip 10.1.1.2      
```

**🌞 Proof !**
```
node1.tp1> show                

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node1.t10.1.1.1/24          0.0.0.0           00:50:79:66:68:00  20002  127.0.0.1:20003
       fe80::250:79ff:fe66:6800/64
```

```
node2.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node2.t10.1.1.2/24          0.0.0.0           00:50:79:66:68:01  20004  127.0.0.1:20005
       fe80::250:79ff:fe66:6801/64
```

**🌞 Effectuer un ping d'une machine à l'autre**

```
node1.tp1> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.097 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.149 ms
84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=0.092 ms
84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=0.094 ms
84 bytes from 10.1.1.2 icmp_seq=5 ttl=64 time=0.086 ms
```

## 5. Analyze

**🌞 Protocolz ?**

Le protocole utilisé est le protocole **ICMP**

**📁 p1_ping.pcap**
voir **p1_ping.pcapng**


# Part 2 : Bring that switch in

**🌞 Déterminer l'adresse MAC de vos trois machines**

```
node1.tp1> show                

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node1.t10.1.1.1/24          0.0.0.0           00:50:79:66:68:00  20002  127.0.0.1:20003
       fe80::250:79ff:fe66:6800/64
```

```
node2.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node2.t10.1.1.2/24          0.0.0.0           00:50:79:66:68:01  20004  127.0.0.1:20005
       fe80::250:79ff:fe66:6801/64
```
```
node3.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node3.t10.1.1.3/24          0.0.0.0           00:50:79:66:68:02  20006  127.0.0.1:20007
       fe80::250:79ff:fe66:6802/64
```

**🌞 Définir une IP statique sur les trois machines**

```
node3.tp1> ip 10.1.1.3
Checking for duplicate address...
node3.tp1 : 10.1.1.3 255.255.255.0

node3.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node3.t10.1.1.3/24          0.0.0.0           00:50:79:66:68:02  20006  127.0.0.1:20007
       fe80::250:79ff:fe66:6802/64
```


**🌞 Effectuer des ping d'une machine à l'autre**

```
node1.tp1> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.071 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.341 ms
84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=0.156 ms
84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=0.254 ms
84 bytes from 10.1.1.2 icmp_seq=5 ttl=64 time=0.226 ms

node2.tp1> ping 10.1.1.3

84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.360 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=0.263 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=0.258 ms
84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=0.254 ms
84 bytes from 10.1.1.3 icmp_seq=5 ttl=64 time=0.359 ms

node1.tp1> ping 10.1.1.3

84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.222 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=0.206 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=0.306 ms
84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=0.228 ms
84 bytes from 10.1.1.3 icmp_seq=5 ttl=64 time=0.286 ms

```


**🌞 Afficher la table ARP de node1**
```
node1.tp1> arp

00:50:79:66:68:02  10.1.1.3 expires in 40 seconds 
00:50:79:66:68:01  10.1.1.2 expires in 23 seconds 
```

**📁 p2_arp_node2.pcap**

**📁 p2_arp_node3.pcap**

# Part 3 : DHCP is a nice guy

**🌞 Installer un serveur DHCP**

```
[amir@dhcp ~]$ vi /etc/dnsmasq.conf
```
ensuite dans la j'ai caller ca:

```
dhcp-range=10.1.1.10,10.1.1.50,12h
interface=enp0s3
```
j'ai vu qlq mettre "interface**s**" et pa "interface", what does it change

```
[amir@dhcp ~]$ firewall-cmd --add-service=dhcp
[amir@dhcp ~]$ firewall-cmd --runtime-to-permanent 
```


**🌞 Récupérer une IP automatiquement depuis les 3 nodes**
```
node3.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node3.t0.0.0.0/0            0.0.0.0           00:50:79:66:68:00  10008  127.0.0.1:10009
       fe80::250:79ff:fe66:6800/64

node3.tp1> dhcp
DDORA IP 10.1.1.45/24 GW 10.1.1.253

node3.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node3.t10.1.1.45/24         10.1.1.253        00:50:79:66:68:00  10008  127.0.0.1:10009
       fe80::250:79ff:fe66:6800/64
```
```
node2.tp1> dhcp
DDORA IP 10.1.1.46/24 GW 10.1.1.253
```
```
node2.tp1> dhcp
DDORA IP 10.1.1.47/24 GW 10.1.1.253

node2.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node2.t10.1.1.47/24         10.1.1.253        00:50:79:66:68:02  10006  127.0.0.1:10007
```
wow enfin, mais ca marche

**🌞 Bail DHCP**

```
[amir@dhcp lib]$ sudo vi /etc/dnsmasq.conf
[sudo] password for amir: 
[amir@dhcp lib]$ cat /var/lib/dnsmasq/dnsmasq.leases 
1767755338 00:50:79:66:68:02 10.1.1.47 node1 01:00:50:79:66:68:02
1767753999 00:50:79:66:68:01 10.1.1.46 node2 01:00:50:79:66:68:01
1767754010 00:50:79:66:68:00 10.1.1.45 node3 01:00:50:79:66:68:00
```

**🌞 Use grep**

```
[amir@dhcp lib]$ cat /var/lib/dnsmasq/dnsmasq.leases | grep node1
1767755338 00:50:79:66:68:02 10.1.1.47 node1 01:00:50:79:66:68:02
```

# Part 4 : real haxor

**🌞 Installez et configurez un serveur DHCP sur votre machine attaquante**
```
┌──(amir㉿amirhost)-[~]
└─$ sudo cat /etc/dnsmasq.conf | grep 'dhcp-range=10'
dhcp-range=10.1.1.210,10.1.1.250,12h
```

**🌞 Test !**
on stop le service sur le "vrai" serveur dhcp
```
[amir@dhcp lib]$ sudo systemctl status dnsmasq | head -n 3
○ dnsmasq.service - DNS caching server.
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; disabled; preset: disabled)
     Active: inactive (dead)
```
sur node1.tp1.efrei:
```
node2.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node2.t10.1.1.46/24         10.1.1.253        00:50:79:66:68:01  10010  127.0.0.1:10011
       fe80::250:79ff:fe66:6801/64

node2.tp1> clear ip
IPv4 address/mask, gateway, DNS, and DHCP cleared

node2.tp1> dhcp
DDORA IP 10.1.1.246/24 GW 10.1.1.252
```


**Race !**

Déjà, on va suppr l'ip de node3
```
node3.tp1> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node3.t10.1.1.45/24         10.1.1.253        00:50:79:66:68:00  10008  127.0.0.1:10009
       fe80::250:79ff:fe66:6800/64

node3.tp1> clear ip
IPv4 address/mask, gateway, DNS, and DHCP cleared
```

Ensuite, on relance le "vrai" dhcp
```
[amir@dhcp lib]$ sudo systemctl start dnsmasq
[sudo] password for amir: 
[amir@dhcp lib]$ sudo systemctl status dnsmasq
● dnsmasq.service - DNS caching server.
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-01-06 17:40:15 CET; 5s ago
```

**📁 p4_dhcp_race.pcap**