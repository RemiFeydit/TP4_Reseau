# TP 4 - Spéléologie réseau : descente dans les couches

# I. Mise en place du Lab

## 1. Création des VMs

* `client1` ping `router1.tp4` sur l'IP `10.1.0.254`
```bash
[remi@client1 ~]$ ping router1.tp4
PING router1 (10.1.0.254) 56(84) bytes of data.
64 bytes from router1 (10.1.0.254): icmp_seq=1 ttl=64 time=0.449 ms
64 bytes from router1 (10.1.0.254): icmp_seq=2 ttl=64 time=0.571 ms
64 bytes from router1 (10.1.0.254): icmp_seq=3 ttl=64 time=0.564 ms
64 bytes from router1 (10.1.0.254): icmp_seq=4 ttl=64 time=0.578 ms
--- router1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 0.449/0.540/0.578/0.057 ms
```
* `server1` ping `router1.tp4` sur l'IP `10.2.0.254`
```bash
[remi@server1 etc]$ ping router1.tp4
PING router1 (10.2.0.254) 56(84) bytes of data.
64 bytes from router1 (10.2.0.254): icmp_seq=1 ttl=64 time=0.357 ms
64 bytes from router1 (10.2.0.254): icmp_seq=2 ttl=64 time=0.447 ms
64 bytes from router1 (10.2.0.254): icmp_seq=3 ttl=64 time=0.416 ms
64 bytes from router1 (10.2.0.254): icmp_seq=4 ttl=64 time=0.551 ms
^C
--- router1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 0.357/0.442/0.551/0.074 ms
```
---
## 2. Mise en place du routage statique

* `ip route show` :
```bash
[remi@router1 ~]$ ip route show
10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.254 metric 100
10.2.0.0/24 dev enp0s9 proto kernel scope link src 10.2.0.254 metric 101
```

* ping `client1` sur `server1`
```bash
[remi@client1 ~]$ ping server1
PING server1 (10.2.0.10) 56(84) bytes of data.
64 bytes from server1 (10.2.0.10): icmp_seq=1 ttl=63 time=0.633 ms
64 bytes from server1 (10.2.0.10): icmp_seq=2 ttl=63 time=1.04 ms
64 bytes from server1 (10.2.0.10): icmp_seq=3 ttl=63 time=0.987 ms
64 bytes from server1 (10.2.0.10): icmp_seq=4 ttl=63 time=0.768 ms
--- server1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 0.633/0.857/1.041/0.166 ms
```
* ping `server1` sur `client1`
```bash
[remi@server1 ~]$ ping client1
PING client1 (10.1.0.10) 56(84) bytes of data.
64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.645 ms
64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=1.12 ms
64 bytes from client1 (10.1.0.10): icmp_seq=3 ttl=63 time=0.775 ms
64 bytes from client1 (10.1.0.10): icmp_seq=4 ttl=63 time=1.16 ms
64 bytes from client1 (10.1.0.10): icmp_seq=5 ttl=63 time=1.10 ms
--- client1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 0.645/0.961/1.160/0.209 ms
```
---
# II. Spéléologie du réseau

# 1. ARP

## A. Manip 1
* Table ARP de `client1`
```bash
[remi@client1 ~]$ ip neigh show
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:46 REACHABLE
```
On a qu'une seule ligne ici car on a vidé la table ARP et qu'aucun ping n'a été fait donc aucune adresse MAC a été ajouté.

* Table ARP de `server1`
```bash
[remi@server1 ~]$ ip neigh show
10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:4b REACHABLE
```
Pareil que pour la Table ARP de `client1`

* Table ARP de `client1` après un ping sur `server1`
```bash
[remi@client1 ~]$ ip neigh show
10.1.0.254 dev enp0s8 lladdr 08:00:27:55:69:2b REACHABLE
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:46 REACHABLE
```
Un ligne a été ajouté, elle est apparu à cause du ping effectué sur `server1`, elle correspond à l'adresse MAC de `server1`

* Table ARP de `server1` après un ping de `client1`
```bash
[remi@server1 ~]$ ip neigh show
10.2.0.254 dev enp0s8 lladdr 08:00:27:9c:42:37 STALE
10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:4b DELAY
```
Une ligne a été ajouté suite au ping de `client1` sur le `server1`

## B. Manip 2

* Table ARP de `router1`
```bash
[remi@router1 /]$ ip neigh show
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:46 REACHABLE
```
Il n'y a qu'une seule ligne car le `router1` n'a communiqué avec aucune autre adresse MAC donc elles ne sont pas dans la table ARP.

* Table ARP de `router1`après un ping de `client1`sur `server1`
```bash
[remi@router1 /]$ ip neigh show
10.1.0.10 dev enp0s8 lladdr 08:00:27:96:37:a3 REACHABLE
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:46 DELAY
10.2.0.10 dev enp0s9 lladdr 08:00:27:fb:54:90 REACHABLE
```
Suite au ping de `client1`sur `server1`, le `router1` servant d'intermédiaire, l'ip MAC de `client1`et `server1` sont enregistré par `router1`

## C. Manip 3 
* Table ARP de l'hôte
```bash
PS C:\Users\Notitou> arp -a

Interface : 10.1.0.1 --- 0x46
  Adresse Internet      Adresse physique      Type
  10.1.0.10             08-00-27-96-37-a3     dynamique
  10.1.0.254            08-00-27-55-69-2b     dynamique
  10.1.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.2.0.1 --- 0x4b
  Adresse Internet      Adresse physique      Type
  10.2.0.10             08-00-27-fb-54-90     dynamique
  10.2.0.254            08-00-27-9c-42-37     dynamique
  10.2.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```
* Table ARP de l'hôte après avoir été vidé
```bash
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.101.1 --- 0xc
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.3.204 --- 0x1a
  Adresse Internet      Adresse physique      Type
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.1.0.1 --- 0x46
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.2.0.1 --- 0x4b
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
```

* Table ARP de l'hôte après une petite attente
```bash
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.101.1 --- 0xc
  Adresse Internet      Adresse physique      Type
  192.168.101.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.33.3.204 --- 0x1a
  Adresse Internet      Adresse physique      Type
  10.33.0.196           f8-59-71-83-5e-78     dynamique
  10.33.1.18            2c-20-0b-41-55-95     dynamique
  10.33.1.135           6c-e8-5c-3d-f4-85     dynamique
  10.33.1.149           dc-a2-66-21-d4-7d     dynamique
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  10.33.3.255           ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.1.0.1 --- 0x46
  Adresse Internet      Adresse physique      Type
  10.1.0.10             08-00-27-96-37-a3     dynamique
  10.1.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.2.0.1 --- 0x4b
  Adresse Internet      Adresse physique      Type
  10.2.0.10             08-00-27-fb-54-90     dynamique
  10.2.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

## D. Manip 4

* Table ARP de `client1`
```bash
[remi@client1 network-scripts]$ ip neigh show
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:46 DELAY
```
* Activation de la carte NAT sur le `client1`
```bash
[remi@client1 network-scripts]$ sudo ifup enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
```
* Table ARP de `client1` après un `curl google.com`
```bash
[remi@client1 network-scripts]$ ip neigh show
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:46 REACHABLE
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
```

* Tableau récapitulatif

Machine | `net1` | `net2` | Adresse MAC
--- | --- | --- | ---
`client1.tp4` | `10.1.0.10` | X | `0a:00:27:00:00:46`
`router1.tp4` | `10.1.0.254` | `10.2.0.254` | `08:00:27:9c:42:37`
`server1.tp4` | X | `10.2.0.10` | `0a:00:27:00:00:4b`
---
# 2.Wireshark

## A. Interception d'ARP et `ping`

* Envoi de 4 ping du `client1`au `server1`

```bash
[remi@client1 ~]$ ping -c 4 server1
PING server1 (10.2.0.10) 56(84) bytes of data.
64 bytes from server1 (10.2.0.10): icmp_seq=1 ttl=63 time=0.591 ms
64 bytes from server1 (10.2.0.10): icmp_seq=2 ttl=63 time=1.08 ms
64 bytes from server1 (10.2.0.10): icmp_seq=3 ttl=63 time=1.00 ms
64 bytes from server1 (10.2.0.10): icmp_seq=4 ttl=63 time=1.05 ms

--- server1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 0.591/0.934/1.088/0.202 ms
```
* Envoi fichier `ping.pcap`de la VM `router1` à l'hôte

```bash
PS C:\Users\Notitou> scp remi@10.2.0.254:/home/remi/ping.pcap C:\Users\Notitou\Desktop\
remi@10.2.0.254's password:
ping.pcap                                                                             100% 1070     1.0KB/s   00:00
```
## B. Interception d'une communication `netcat`

* `SYN`
```wireshark
"7","34.700033","10.1.0.10","10.2.0.10","TCP","74","42470 → 8888 [SYN] Seq=0 Win=29200 Len=0 MSS=1460 SACK_PERM=1 TSval=3941926 TSecr=0 WS=64"
```
* `SYN, ACK`
```wireshark
"8","34.700244","10.2.0.10","10.1.0.10","TCP","74","8888 → 42470 [SYN, ACK] Seq=0 Ack=1 Win=28960 Len=0 MSS=1460 SACK_PERM=1 TSval=3941300 TSecr=3941926 WS=64"
```

* `ACK`
```wireshark
"9","34.700561","10.1.0.10","10.2.0.10","TCP","66","42470 → 8888 [ACK] Seq=1 Ack=1 Win=29248 Len=0 TSval=3941928 TSecr=3941300"
```

* `nop frer`
```wireshark
"150","117.403421","10.2.0.10","10.1.0.10","ICMP","102","Destination unreachable (Host administratively prohibited)"
```