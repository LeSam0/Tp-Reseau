# Tp 4 Réseau 

## I.First steps

**Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

 - Hyperplanning : TCP 
 ip et port srv : 178.32.154.10:443
port lcl : 51501
[TCP Hyperplanning](./Wireshark/tp4_hyperplanningtcp.pcapng)

 - Discord : UDP 
 ip et port srv : 35.214.249.235:50004
port lcl : 51522
[UDP Discord](./Wireshark/tp4_discordudp.pcapng)
 - Youtube : QUIC
ip et port srv : 91.68.245.13:443
port lcl : 52487
[QUIC Youtube](./Wireshark/tp4_youtubequic.pcapng)
 - Steam : TCP
ip et port srv : 104.126.244.161:443
port lcl : 63810
[TCP Steam](./Wireshark/tp4_steamtcp.pcapng)
 - Jeux : UDP
ip et port srv : 
222.131.28.20:3885
port lcl : 53698
[UDP Jeux](./Wireshark/tp4_jeuxudp.pcapng)


**Demandez l'avis à votre OS**

    $ netstat -n -b -a
    Proto  Adresse locale         Adresse distante       État
    [Steam.exe]
    TCP    10.33.17.14:63803      104.126.244.161:443    ESTABLISHED
    [chrome.exe]
    TCP    10.33.17.14:51501      178.32.154.10:443      ESTABLISHED
    [Discord.exe]
    UDP    0.0.0.0:51522          *:*
    [PaintTheTownRed.exe]
    UDP    0.0.0.0:53698          *:*
    [msedgewebview2.exe]
    UDP    0.0.0.0:52487          91.68.245.13:443
    
    
## II. Mise en place

#### *1. SSH*

**Examinez le trafic dans WiresharkP**

C'est du TCP parce qu'on donne des commands et de mot de passe ou autre donc vaut mieux que tous soit dans l'ordre

[SSH](./Wireshark/tp4_ssh.pcapng)

**Demandez aux OS**

    $ netstat -n -b
    [ssh.exe]
    TCP    10.33.17.14:49458      20.199.120.182:443     ESTABLISHED
    WpnService

## III. DNS

#### *2. Setup*

named.conf : 

    $ sudo cat /etc/named.conf
    //
    // named.conf
    //
    // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
    // server as a caching only nameserver (as a localhost DNS resolver only).
    //
    // See /usr/share/doc/bind*/sample/ for example named configuration files.
    //

    options {
        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access

           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
    };

    logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
    };

    zone "tp4.b1" IN {
        type master;
        file "tp4.b1.db";
        allow-update { none; };
        allow-query { any; };
    };

    zone "1.4.10.in-addr.arpa" IN {
        type master;
        file "tp4.b1.rev";
        allow-update { none; };
        allow-query { any; };
    };



    include "/etc/named.rfc1912.zones";
    include "/etc/named.root.key";
    
Tp4.b1.db : 
 
    $ sudo cat /var/named/tp4.b1.db
    $TTL 86400
    @ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
        2019061800 ;Serial
        3600 ;Refresh
        1800 ;Retry
        604800 ;Expire
        86400 ;Minimun TTL
    )

    ; Infos sur le serveur DNS lui même (NS = NameServer)
    @ IN NS dns-server.tp4.b1.

    ; Enregistrements DNS pour faire correspondre des noms à des IPs
    dns-server IN A 10.4.1.201
    node1      IN A 10.4.1.11
    
Tp4.b1.rev : 

    $ sudo cat /var/named/tp4.b1.rev
    $TTL 86400
    @ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
        2019061800; Serial
        3600 ;Refresh
        1800 ;Retry
        604800
        86400;Minimun TTL
    )

    ; Infos sur le serveur DNS lui même (NS = NameServer)
    @ IN NS dns-server.tp4.b1.

    ;Reverse lookup for Name Server
    201 IN PTR dns-serveur.tp4.b1.
    11 IN PTR node1.tp4.b1.
    
Status : 

    $ systemctl status named
    ● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: di>
     Active: active (running) since Thu 2022-11-10 10:27:11 CET; 13min ago
       Main PID: 33192 (named)
      Tasks: 5 (limit: 5906)
     Memory: 16.8M
        CPU: 41ms
     CGroup: /system.slice/named.service
             └─33192 /usr/sbin/named -u named -c /etc/named.conf

    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: network unreachable resolving './DNSKE>
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: network unreachable resolving './NS/IN>
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: zone 1.0.0.127.in-addr.arpa/IN: loaded>
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: zone localhost/IN: loaded serial 0
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: zone localhost.localdomain/IN: loaded >
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: all zones loaded
    Nov 10 10:27:11 dns-server.tp4.b1 systemd[1]: Started Berkeley Internet Name Domain (D>
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: running
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: managed-keys-zone: Initializing automa>
    Nov 10 10:27:11 dns-server.tp4.b1 named[33192]: resolver priming query complete

SS : 

    $ sudo ss -ltupn
    Netid          State           Recv-Q          Send-Q                   Local Address:Port                   Peer Address:Port         Process
    udp            UNCONN          0               0                           10.4.1.201:53                          0.0.0.0:*             users:(("named",pid=33393,fd=19))
    udp            UNCONN          0               0                            127.0.0.1:53                          0.0.0.0:*             users:(("named",pid=33393,fd=16))
    udp            UNCONN          0               0                                [::1]:53                             [::]:*             users:(("named",pid=33393,fd=22))
    tcp            LISTEN          0               10                          10.4.1.201:53                          0.0.0.0:*             users:(("named",pid=33393,fd=21))
    tcp            LISTEN          0               10                           127.0.0.1:53                          0.0.0.0:*             users:(("named",pid=33393,fd=17))
    tcp            LISTEN          0               4096                         127.0.0.1:953                         0.0.0.0:*             users:(("named",pid=33393,fd=24))
    tcp            LISTEN          0               10                               [::1]:53                             [::]:*             users:(("named",pid=33393,fd=23))
    tcp            LISTEN          0               4096                             [::1]:953                            [::]:*             users:(("named",pid=33393,fd=25))
    
Firewall : 

    $ sudo firewall-cmd --add-port=53/tcp --permanent
    success
    $ sudo firewall-cmd --add-port=53/udp --permanent
    success
    $ sudo firewall-cmd --reload
    success
    
    
#### *3. Test*

**Sur la machine node1.tp4.b1**

    $ sudo cat /etc/resolv.conf
    # Generated by NetworkManager
    nameserver 10.4.1.201
    
Google : 

    $ dig www.google.com

    ; <<>> DiG 9.16.23-RH <<>> www.google.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61713
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: beba228f2fbd5e4701000000636cd2f0299fcd6cc6bd998c (good)
    ;; QUESTION SECTION:
    ;www.google.com.                        IN      A

    ;; ANSWER SECTION:
    www.google.com.         300     IN      A       216.58.214.164

    ;; Query time: 399 msec
    ;; SERVER: 10.4.1.201#53(10.4.1.201)
    ;; WHEN: Thu Nov 10 11:31:12 CET 2022
    ;; MSG SIZE  rcvd: 87
    
node1.tp4.b1 : 
    
    $ dig node1.tp4.b1

    ; <<>> DiG 9.16.23-RH <<>> node1.tp4.b1
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20000
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: b57394d85f293dcc01000000636cd3535be98feb5265a06d (good)
    ;; QUESTION SECTION:
    ;node1.tp4.b1.                  IN      A

    ;; ANSWER SECTION:
    node1.tp4.b1.           86400   IN      A       10.4.1.11

    ;; Query time: 0 msec
    ;; SERVER: 10.4.1.201#53(10.4.1.201)
    ;; WHEN: Thu Nov 10 11:32:51 CET 2022
    ;; MSG SIZE  rcvd: 85

dns-server.tp4.b1 : 

    $ dig dns-server.tp4.b1

    ; <<>> DiG 9.16.23-RH <<>> dns-server.tp4.b1
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59922
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: d8fe4ca706cd796a01000000636cd39ab25068c297339281 (good)
    ;; QUESTION SECTION:
    ;dns-server.tp4.b1.             IN      A

    ;; ANSWER SECTION:
    dns-server.tp4.b1.      86400   IN      A       10.4.1.201

    ;; Query time: 0 msec
    ;; SERVER: 10.4.1.201#53(10.4.1.201)
    ;; WHEN: Thu Nov 10 11:34:02 CET 2022
    ;; MSG SIZE  rcvd: 90
    
**Sur votre PC**

    PS C:\Users\samyd> nslookup node1.tp4.b1 10.4.1.201
    Serveur :   dns-serveur.tp4.b1
    Address:  10.4.1.201

    Nom :    node1.tp4.b1
    Address:  10.4.1.11

[DNS](./Wireshark/tp4_digpc.pcapng)