# TP 2

## Part1 : Network Setup

**🌞 Tout le monde doit pouvoir se ping**

tout le monde peut se ping dans leurs deux LANs respectifs:

**node1** vers **node2**
```
node1.tp2> ping 10.2.1.12

84 bytes from 10.2.1.12 icmp_seq=1 ttl=64 time=0.913 ms
84 bytes from 10.2.1.12 icmp_seq=2 ttl=64 time=0.389 ms
84 bytes from 10.2.1.12 icmp_seq=3 ttl=64 time=0.448 ms
84 bytes from 10.2.1.12 icmp_seq=4 ttl=64 time=0.379 ms
84 bytes from 10.2.1.12 icmp_seq=5 ttl=64 time=0.717 ms
```
**node3** vers **node4**
```
node3.tp2> ping 10.2.2.12

84 bytes from 10.2.2.12 icmp_seq=1 ttl=64 time=0.479 ms
84 bytes from 10.2.2.12 icmp_seq=2 ttl=64 time=0.179 ms
84 bytes from 10.2.2.12 icmp_seq=3 ttl=64 time=0.324 ms
84 bytes from 10.2.2.12 icmp_seq=4 ttl=64 time=0.238 ms
84 bytes from 10.2.2.12 icmp_seq=5 ttl=64 time=0.223 ms
```
et le routage fonctionne:

**node1** vers **node3**
```
node1.tp2> ping 10.2.2.11

84 bytes from 10.2.2.11 icmp_seq=1 ttl=63 time=29.553 ms
84 bytes from 10.2.2.11 icmp_seq=2 ttl=63 time=16.267 ms
84 bytes from 10.2.2.11 icmp_seq=3 ttl=63 time=17.474 ms
84 bytes from 10.2.2.11 icmp_seq=4 ttl=63 time=16.556 ms
84 bytes from 10.2.2.11 icmp_seq=5 ttl=63 time=17.840 ms
```
**node4** vers **node2**
```
node4.tp2> ping 10.2.1.12

84 bytes from 10.2.1.12 icmp_seq=1 ttl=63 time=29.787 ms
84 bytes from 10.2.1.12 icmp_seq=2 ttl=63 time=17.822 ms
84 bytes from 10.2.1.12 icmp_seq=3 ttl=63 time=18.992 ms
84 bytes from 10.2.1.12 icmp_seq=4 ttl=63 time=12.830 ms
84 bytes from 10.2.1.12 icmp_seq=5 ttl=63 time=17.982 ms
```

**📁 p1_routed_ping.pcap** dans l'dossier TP2


## Part2 : Network Setup

**🌞 Prouver que...**

deja paf, on a une interface avec pas d'ip
```
R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.2.1.254      YES manual up                    up
FastEthernet1/0            10.2.2.254      YES manual up                    up
FastEthernet2/0            unassigned      YES unset  administratively down down
```

ensuite, en suivant la magnifique doc, on lance la commande ```ip address dhcp``` dans 2/0:
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface fastethernet2/0
R1(config-if)#ip address dhcp
R1(config-if)#no shut
```

et on verifie:
```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.2.1.254      YES manual up                    up
FastEthernet1/0            10.2.2.254      YES manual up                    up
FastEthernet2/0            192.168.122.114 YES DHCP   up                    up
```

et maintenant on essaye de ping:
```
R1#ping ip 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
.U.U.
Success rate is 0 percent (0/5)
```
avant de se rendre compte qu'on a pas ajouter de route par défaut, donc le routeur ne savait pas où se trouvait "1.1.1.1" donc il faisait rien, logique...
on fix ca:
```
R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.122.1
```

et maintenant on ping:
```
R1#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/18/20 ms
```

ping de node1 vers 1.1.1.1:
**📁 p1_no_nat.pcap**

**🌞 Proooooooooof or lie**
configurer le nat, j'me réfère au mémo cisco pour ca
interface externe c'est celle qui pointe vers internet, et interne vers les lans

déjà, on regarde nos interfaces et leurs IPs, pour savoir qui pointe où
```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.2.1.254      YES manual up                    up
FastEthernet1/0            10.2.2.254      YES manual up                    up
FastEthernet2/0            192.168.122.114 YES DHCP   up                    up
```

ensuite bah, on conf les interfaces, les 3, dans le but que quand node1 ping vers internet le routeur fait passer le ping avec l'adresse IP de l'interface qui pointe vers le NAT, et ensuite il redirige la réponse vers 
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface fastethernet2/0
R1(config-if)#ip nat outside

*Jan 13 12:53:02.615: %LINEPROTO-5-UPDOWN: Line protocol on Interface NVI0, changed state to up
R1(config-if)#exit
R1(config)#interface fastethernet0/0
R1(config-if)#ip nat inside
R1(config-if)#exit
R1(config)#interface fastethernet1/0
R1(config-if)#ip nat inside
R1(config-if)#exit
R1(config)#access-list 1 permit any
R1(config)#ip nat inside source list 1 interface fastethernet2/0 overload
```

et puis paf, on peut ping avec node1:
```
node1.tp2> ping 1.1.1.1
84 bytes from 1.1.1.1 icmp_seq=1 ttl=54 time=29.872 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=54 time=22.425 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=54 time=31.272 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=54 time=31.281 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=54 time=61.733 ms
```

**📁 r1_running_config.txt**

### 3. Vrai accès internet clients

**🌞 Prove it**
```
node1.tp2> ip dns 1.1.1.1

node1.tp2> ping efrei.fr
efrei.fr resolved to 51.210.229.203
84 bytes from 51.210.229.203 icmp_seq=1 ttl=48 time=29.368 ms
84 bytes from 51.210.229.203 icmp_seq=2 ttl=48 time=31.236 ms
84 bytes from 51.210.229.203 icmp_seq=3 ttl=48 time=22.519 ms
84 bytes from 51.210.229.203 icmp_seq=4 ttl=48 time=21.939 ms
84 bytes from 51.210.229.203 icmp_seq=5 ttl=48 time=22.416 ms
```

**🌞 Test test test : ajouter un nouveau VPCS au LAN1, le bro node5.tp2.efrei**

```
node5.tp2> dhcp
DDORA IP 10.2.1.164/24 GW 10.2.1.254

node5.tp2> show

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
node5.t10.2.1.164/24        10.2.1.254        00:50:79:66:68:04  10037  127.0.0.1:10038
       fe80::250:79ff:fe66:6804/64

node5.tp2> ping 1.1.1.1
84 bytes from 1.1.1.1 icmp_seq=1 ttl=54 time=29.335 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=54 time=21.972 ms

node5.tp2> ping efrei.fr
efrei.fr resolved to 51.210.229.203
84 bytes from 51.210.229.203 icmp_seq=1 ttl=48 time=29.273 ms
84 bytes from 51.210.229.203 icmp_seq=2 ttl=48 time=31.248 ms
```
## Part3 : Time to attack all this

### ARP spoofing
**📁 p3_arp_mitm.pcapng**

### DHCP spoofing
**🌞 Test test test : ajouter un nouveau VPCS au LAN1, le bro node6.tp2.efrei**

premiere etape, node demande une ip, notre rogue dhcp serv il répond en mode "yo c moi c le vrai" (de toute façon les deux serveurs dhcp sont des vrais, juste un des deux est utilisé à des fins méchantes quoi...)
```
node6> dhcp
DDORA IP 10.2.1.240/24 GW 10.2.1.250
```

et il a internet en plus, trop cool non?
```
node6> ping 1.1.1.1
84 bytes from 1.1.1.1 icmp_seq=1 ttl=54 time=39.767 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=54 time=32.289 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=54 time=22.389 ms
```

## Part4 : Alors koa c tou ? On refé just la mem choz ke o tp1 enfet enfet ? Bah non

**🌞 Préparer le DNS spoof**
pas trooop sur de quoi mettre ici, car j'ai copier coller ce que tu mets, fin tu nous donnes la conf quoi, bah j'ai ```firewall-cmd --permanent add-```

**🌞 S'assurer que c'est up & running, on en profite pour réviser un peu de shell**

```
ps -ef | grep dnsmasq | head -n 1
dnsmasq     1503       1  0 20:27 ?        00:00:00 dnsmasq -C /tmp/dnsmasq.conf -q
```

```
sudo ss -lnpu | grep dnsmasq
UNCONN 0      0            0.0.0.0:53        0.0.0.0:*    users:(("dnsmasq",pid=1503,fd=4))
UNCONN 0      0               [::]:53           [::]:*    users:(("dnsmasq",pid=1503,fd=6))
```

**🌞 Relance ton attaque DHCP spoof depuis la machine attaquante**
```
dhcp-option=6,10.2.1.250
```
6 pour indiquer dns, c'est le numéro de l'option

** **
```
node7> dhcp
DDORA IP 10.2.1.163/24 GW 10.2.1.254

node7> show ip

NAME        : node7[1]
IP/MASK     : 10.2.1.163/24
GATEWAY     : 10.2.1.254
DNS         : 10.2.1.250
DHCP SERVER : 10.2.1.250
DHCP LEASE  : 42997, 43200/21600/37800
MAC         : 00:50:79:66:68:03
LPORT       : 10037
RHOST:PORT  : 127.0.0.1:10038
MTU:        : 1500

node7> ping 1.1.1.1
84 bytes from 1.1.1.1 icmp_seq=1 ttl=54 time=29.537 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=54 time=26.538 ms

node7> ping efrei.fr
efrei.fr resolved to 51.210.229.203
84 bytes from 51.210.229.203 icmp_seq=1 ttl=48 time=29.444 ms
84 bytes from 51.210.229.203 icmp_seq=2 ttl=48 time=25.438 ms
```