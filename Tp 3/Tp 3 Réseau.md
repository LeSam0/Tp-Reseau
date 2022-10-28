# Tp 3 Réseau 

## I.ARP

#### *1. Echange ARP*

**Générer des requêtes ARP**

**PING :** 

`ping 10.3.1.11` ou `ping 10.3.1.12`

john

    PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
    64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.235 ms
    64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.244 ms
    64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.268 ms
    64 bytes from 10.3.1.11: icmp_seq=4 ttl=64 time=0.393 ms
    64 bytes from 10.3.1.11: icmp_seq=5 ttl=64 time=0.216 ms
    --- 10.3.1.11 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4082ms
    rtt min/avg/max/mdev = 0.216/0.271/0.393/0.063 ms
    
marcel

    PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
    64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.189 ms
    64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.344 ms
    64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.251 ms
    64 bytes from 10.3.1.12: icmp_seq=4 ttl=64 time=0.247 ms
    64 bytes from 10.3.1.12: icmp_seq=5 ttl=64 time=0.224 ms
    64 bytes from 10.3.1.12: icmp_seq=6 ttl=64 time=0.247 ms
    64 bytes from 10.3.1.12: icmp_seq=7 ttl=64 time=0.215 ms
    64 bytes from 10.3.1.12: icmp_seq=8 ttl=64 time=0.340 ms
    --- 10.3.1.12 ping statistics ---
    8 packets transmitted, 8 received, 0% packet loss, time 7157ms
    rtt min/avg/max/mdev = 0.189/0.257/0.344/0.052 ms

**ARP :**

john

    ip neigh show
    10.3.1.12 dev enp0s8 lladdr 08:00:27:e4:ad:f3 STALE
    10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3d REACHABLE

marcel

    ip neigh show
    10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3d REACHABLE
    10.3.1.11 dev enp0s8 lladdr 08:00:27:b5:35:46 STALE
    
Addresse MAC de John : 08:00:27:b5:35:46
Addresse MAC de Marcel : 08:00:27:e4:ad:f3

MAC de Marcel depuis John : 
`ip neigh show`
10.3.1.12 dev enp0s8 lladdr 08:00:27:e4:ad:f3 STALE

MAC de Marcel depuis Marcel : 
`ip a`
link/ether 08:00:27:e4:ad:f3 brd ff:ff:ff:ff:ff:ff


#### *2. Analyse de trames*

**Analyse de trames**

`sudo tcpdump -i enp0s8 -c 10 -w tp3_arp.pcapng not port 22`

`sudo ip neigh flush all
`
`ping 10.3.1.12`


[ARP](./Wireshark/tp3_arp.pcapng)


## II. Routage

#### *1. Mise en place du routage*

**Activer le routage sur le noeud router**


`$ sudo firewall-cmd --list-all`
`$ sudo firewall-cmd --get-active-zone`

`$ sudo firewall-cmd --add-masquerade --zone=public`
`$ sudo firewall-cmd --add-masquerade --zone=public --permanent`


**Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping**

John : 

    $ sudo nano /etc/sysconfig/network-scripts/route-enp0s8
    10.3.2.12/24 via 10.3.1.254 dev enth0
    $ sudo systemctl restart NetworkManager
    $ ip route show
    10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.11 metric 100
    10.3.2.0/24 via 10.3.1.254 dev enp0s8 proto static metric 100
    $ ping 10.3.2.12
    PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
    64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.455 ms
    64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.447 ms
    64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.549 ms
    64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=0.366 ms
    --- 10.3.2.12 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3103ms
    rtt min/avg/max/mdev = 0.366/0.454/0.549/0.064 ms
    
Marcel : 

     $ sudo nano /etc/sysconfig/network-scripts/route-enp0s8
      10.3.2.12/24 via 10.3.1.254 dev enth0
    $ sudo systemctl restart NetworkManager
    $ ip route show
    10.3.1.0/24 via 10.3.2.254 dev enp0s8 proto static metric 100
    10.3.2.0/24 dev enp0s8 proto kernel scope link src 10.3.2.12 metric 100
    $ ping 10.3.1.11
    PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
    64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=0.728 ms
    64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=0.433 ms
    64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=0.415 ms
    --- 10.3.1.11 ping statistics ---
    13 packets transmitted, 13 received, 0% packet loss, time 12300ms
    rtt min/avg/max/mdev = 0.393/0.498/0.759/0.111 ms
    

#### *2. Analyse de trames*

**Analyse des échanges ARP**

John : 

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | 10.3.1.11 |`john` `08:00:27:b5:35:46`| 10.3.1.254    |`routeur` `08:00:27:b5:35:46`|
| 2     | Réponse ARP | 10.3.1.254|`routeur` `08:00:27:b5:35:46`| 10.3.1.11  |`john``08:00:27:b5:35:46`   |
| ...   | ...         | ...       | ...                     |                |                            |
| ?     | Ping        | 10.3.1.11 |`john` `08:00:27:b5:35:46`|10.3.2.12      |`marcel` `08:00:27:e4:ad:f3`|
| ?     | Pong        |10.3.2.12  |`marcel` `08:00:27:e4:ad:f3`|10.3.1.11    |`john` `08:00:27:b5:35:46`  |

Marcel : 

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | 10.3.2.12 |`marcel` `08:00:27:e4:ad:f3`|10.3.2.254   |`routeur` `08:00:27:b5:35:46`|
| 2     | Réponse ARP | 10.3.2.254|`routeur` `08:00:27:b5:35:46`| 10.3.2.12  | `marcel` `08:00:27:e4:ad:f3`|
| ...   | ...         | ...       | ...                     |                |                            |
| ?     | Ping        |10.3.2.12  |`marcel` `08:00:27:e4:ad:f3`|10.3.1.11       |`john` `08:00:27:b5:35:46`  |
| ?     | Pong        |10.3.1.11  |`john` `08:00:27:b5:35:46`|10.3.2.12      |`marcel` `08:00:27:e4:ad:f3`|

[ROUTEUR](./Wireshark/tp3_routage_marcel.pcapng)

#### *3. Accès internet*

**Donnez un accès internet à vos machines**

John : 

    $ sudo nano /etc/sysconfig/network
    GATEWAY=10.3.1.254
    $ sudo systemctl restart NetworkManager
    $ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=295 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=157 ms
    --- 8.8.8.8 ping statistics ---
    6 packets transmitted, 6 received, 0% packet loss, time 5010ms
    rtt min/avg/max/mdev = 23.416/131.482/294.556/86.821 ms
    $ sudo nano /etc/resolv.conf
    nameserver 1.1.1.1
    $ dig gitlab.com
    ; <<>> DiG 9.16.23-RH <<>> gitlab.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61183
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ;; QUESTION SECTION:
    ;gitlab.com.                    IN      A

    ;; ANSWER SECTION:
    gitlab.com.             226     IN      A       172.65.251.78

    ;; Query time: 26 msec
    ;; SERVER: 1.1.1.1#53(1.1.1.1)
    ;; WHEN: Tue Oct 25 12:02:19 CEST 2022
    ;; MSG SIZE  rcvd: 55
    $ ping google.com
    PING google.com (142.250.179.110) 56(84) bytes of data.
    64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=247 time=296 ms
    64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=2 ttl=247 time=26.1 ms
    --- google.com ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 25.047/115.852/296.401/127.667 ms
    
Marcel : 

     $ sudo nano /etc/sysconfig/network
    GATEWAY=10.3.2.254
    $ sudo systemctl restart NetworkManager
    $ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=295 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=157 ms
    --- 8.8.8.8 ping statistics ---
    6 packets transmitted, 6 received, 0% packet loss, time 5010ms
    rtt min/avg/max/mdev = 23.416/131.482/294.556/86.821 ms
    $ sudo nano /etc/resolv.conf
    nameserver 1.1.1.1
    $ dig gitlab.com
    ; <<>> DiG 9.16.23-RH <<>> gitlab.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61183
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ;; QUESTION SECTION:
    ;gitlab.com.                    IN      A

    ;; ANSWER SECTION:
    gitlab.com.             226     IN      A       172.65.251.78

    ;; Query time: 26 msec
    ;; SERVER: 1.1.1.1#53(1.1.1.1)
    ;; WHEN: Tue Oct 25 12:02:19 CEST 2022
    ;; MSG SIZE  rcvd: 55
    $ ping google.com
    PING google.com (142.250.179.110) 56(84) bytes of data.
    64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=247 time=296 ms
    64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=2 ttl=247 time=26.1 ms
    --- google.com ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 25.047/115.852/296.401/127.667 ms

**Analyse de trames**

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |
|-------|------------|--------------------|-------------------------|----------------|-----------------|
| 1     | ping       |`john` `10.3.1.12`  |`john` `08:00:27:b5:35:46`| `8.8.8.8`      |`08:00:27:9c:b8:aa`|
| 2     | pong       | `8.8.8.8`          |`08:00:27:9c:b8:aa`      |`john` `10.3.1.12`|`john` `08:00:27:b5:35:46`|

[INTERNET](./Wireshark/tp3_routage_internet.pcapng)

## III. DHCP

#### *1. Mise en place du serveur DHCP*

**Sur la machine john, vous installerez et configurerez un serveur DHCP**

John : 

    $ dnf -y install dhcp-server
    $ sudo nano /etc/dhcp/dhcpd.conf
    # create new
    # specify DNS server's hostname or IP address
    option domain-name-servers     1.1.1.1;
    # default lease time
    default-lease-time 600;
    # max lease time
    max-lease-time 7200;
    # this DHCP server to be declared valid
    authoritative;
    # specify network address and subnetmask
    subnet 10.3.1.0 netmask 255.255.255.0 {
        # specify the range of lease IP address
        range dynamic-bootp 10.3.1.2 10.3.1.253;
        # specify broadcast address
        option broadcast-address 10.3.1.255;
        # specify gateway
        option routers 10.3.1.254;
    }

    $ sudo systemctl enable --now dhcpd
    $ sudo firewall-cmd --add-service=dhcp
    success
    $ sudo firewall-cmd --runtime-to-permanent
    success

Bob : 

    $ sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s8 
    DEVICE=enp0s8

    BOOTPROTO=dhcp
    ONBOOT=yes
    $ sudo nmcli con reload
    $ sudo nmcli con up "System enp0s8"
    $ sudo systemctl restart NetworkManager


**Améliorer la configuration du DHCP**

John : 

    $ sudo nano /etc/dhcp/dhcpd.conf
    
    option domain-name-servers     1.1.1.1;
    option routers 10.3.1.254;
    
    $ sudo systemctl enable --now dhcpd

Marcel:

    $ sudo ip addr del 10.3.1.2/24 dev enp0s8
    $ ip a 
    
    2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:19:47:6e brd ff:ff:ff:ff:ff:ff
    inet6 fe80::a00:27ff:fe19:476e/64 scope link
       valid_lft forever preferred_lft forever
       
    $ sudo systemctl restart NetworkManager
    
    2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:19:47:6e brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 545sec preferred_lft 545sec
    inet6 fe80::a00:27ff:fe19:476e/64 scope link
       valid_lft forever preferred_lft forever

    $ ping 10.3.1.254
    
    PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
    64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.264 ms
    --- 10.3.1.254 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 2640ms
    rtt min/avg/max/mdev = 0.243 ms
     
    $ ip route show
    
    default via 10.3.1.254 dev enp0s8 proto dhcp src 10.3.1.2 metric 100
    10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.2 metric 100
    
    $ ping 10.3.1.11
    
    PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
    64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.200 ms
    64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.191 ms
    --- 10.3.1.11 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1004ms
    rtt min/avg/max/mdev = 0.191/0.195/0.200/0.004 ms
    
    $ dig 1.1.1.1

    ; <<>> DiG 9.16.23-RH <<>> 1.1.1.1
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 54695
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ;; QUESTION SECTION:
    ;1.1.1.1.                       IN      A

    ;; AUTHORITY SECTION:
    .                       85990   IN      SOA     a.root-servers.net. nstld.verisign-grs.com. 2022102800 1800 900 604800 86400

    ;; Query time: 130 msec
    ;; SERVER: 1.1.1.1#53(1.1.1.1)
    ;; WHEN: Fri Oct 28 10:09:23 CEST 2022
    ;; MSG SIZE  rcvd: 111
    
    $ ping google.com
    
    PING google.com (216.58.214.174) 56(84) bytes of data.
    64 bytes from mad01s26-in-f14.1e100.net (216.58.214.174): icmp_seq=1 ttl=247 time=331 ms
    64 bytes from mad01s26-in-f14.1e100.net (216.58.214.174): icmp_seq=2 ttl=247 time=20.0 ms
    --- google.com ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1000ms
    rtt min/avg/max/mdev = 19.960/175.610/331.261/155.650 ms

#### *2. Analyse de trames*

**Analyse de trames**

John : 

    $ sudo nano /etc/dhcp/dhcpd.conf
    
    range dynamic-bootp 10.3.1.232 10.3.1.253;
    
    $ sudo systemctl disable --now dhcpd  
    $ sudo systemctl enable --now dhcpd
    
Marcel : 
    
    $ sudo tcpdump -i enp0s8 -w tp3_dhcp.pcapng &
    $ sudo ip addr del 10.3.1.2/24 dev enp0s8
    $ sudo nmcli con down "System enp0s8"
    $ sudo nmcli con up "System enps0s8"
    

l'IP fournie au client : `Your (client) IP address: 10.3.1.232`
l'adresse IP de la passerelle : `Router: 10.3.1.254`
l'adresse du serveur DNS que vous proposez au client : `Domain Name Server: 1.1.1.1`
    
[DHCP](./Wireshark/tp3_dhcp.pcapng)