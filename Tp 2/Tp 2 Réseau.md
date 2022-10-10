# Tp 2 Réseau

## I. Setup IP

**Mettez en place une configuration réseau fonctionnelle entre les deux machines**

ip pc1 : 10.10.27.65
ip pc2 : 10.10.27.66

l'adresse de réseau :  10.10.24.0/22

l'adresse de broadcast : 10.10.27.255/22

    netsh interface ip set address name=“Ethernet” static 10.10.27.65 255.255.252.0
    
**Prouvez que la connexion est fonctionnelle entre les deux machines**

     ping 10.10.27.66

Envoi d’une requête 'Ping'  10.10.27.66 avec 32 octets de données :
Réponse de 10.10.27.66 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.27.66 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.27.66 : octets=32 temps=3 ms TTL=128
Réponse de 10.10.27.66 : octets=32 temps=3 ms TTL=128

Statistiques Ping pour 10.10.27.66:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 3ms, Moyenne = 2ms

**Wireshark it**

[ping](./Wireshark/ping.pcapng)

Pour le ping envoie est un ICMP de type 
   
   Type: 8 (Echo (ping) request)
    
Pour le ping retour est un ICMP de type 

Type: 0 (Echo (ping) reply)

## II. ARP my bro

**Check the ARP table**

    arp -a
la MAC de mon binome est 10.10.27.66           b4-45-06-96-86-c2     dynamique

la MAC de la gateway est   10.33.19.254          00-c0-e7-e0-04-4e     dynamique
    
**Manipuler la table ARP**

    netsh interface ip delete arpcache
**Avant**

    Interface : 10.10.27.65 --- 0xe
      Adresse Internet      Adresse physique      Type
      10.10.27.66           b4-45-06-96-86-c2     dynamique
      10.10.27.255          ff-ff-ff-ff-ff-ff     statique
      224.0.0.2             01-00-5e-00-00-02     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.251           01-00-5e-00-00-fb     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
      255.255.255.255       ff-ff-ff-ff-ff-ff     statique`
    
**Après**
 
    Interface : 10.10.27.65 --- 0xe
      Adresse Internet      Adresse physique      Type
      224.0.0.2             01-00-5e-00-00-02     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      255.255.255.255       ff-ff-ff-ff-ff-ff     statique`

**Après Ping**

    Interface : 10.10.27.65 --- 0xe
    Adresse Internet      Adresse physique      Type
      10.10.27.66           b4-45-06-96-86-c2     dynamique
      10.10.27.255          ff-ff-ff-ff-ff-ff     statique
      224.0.0.2             01-00-5e-00-00-02     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.251           01-00-5e-00-00-fb     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
      255.255.255.255       ff-ff-ff-ff-ff-ff     statique

**Wireshark it**

[ARP](./Wireshark/arp.pcapng)

ARP broadcast

    Destination: Broadcast (ff:ff:ff:ff:ff:ff)
    Source: (MON PC) ASUSTekC_89:b3:1a (04:42:1a:89:b3:1a)
    
ARP reply

    Destination: (MON PC)  ASUSTekC_89:b3:1a )(04:42:1a:89:b3:1a)
    Source: (PC 2) Dell_96:86:c2 (b4:45:06:96:86:c2)

## II.5 Interlude hackerzz

## III. DHCP you too my brooo

**Wireshark it**

[DORA](./Wireshark/DORA.pcapng)

- 1. une IP à utiliser : 

`10.33.17.14`

- 2. l'adresse IP de la passerelle du réseau

`10.33.19.254`

- 3. l'adresse d'un serveur DNS joignable depuis ce réseau

`DNS: 8.8.8.8`
`DNS: 1.1.1.1`
`DNS: 8.8.4.4`


pour le D : 
- source 0.0.0.0
- dest 255.255.255.255

pour le O : 
- source 10.33.19.254
- dest 255.255.255.255

pour le R :
- source 0.0.0.0
- dest 255.255.255.255

pour le A : 
- source 10.33.19.254
- dest 255.255.255.255

(je chercher pk tous est en broadcast , en gros le D et le R est normal, mais le A est peter et le O, 1 fois sur 14 ca ce met en broadcast. En gros c'est peter, et seul mon pc a 2 400 euro a ce problème ça vallait le coup de payer autant. Merci ASUS <3 :100: )

