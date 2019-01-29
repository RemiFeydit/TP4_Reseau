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

Interface : 192.168.101.1 --- 0xc
  Adresse Internet      Adresse physique      Type
  192.168.101.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  226.43.168.192        01-00-5e-2b-a8-c0     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.33.3.204 --- 0x1a
  Adresse Internet      Adresse physique      Type
  10.33.0.5             e4-a4-71-cb-56-56     dynamique
  10.33.0.6             b0-70-2d-b8-9f-24     dynamique
  10.33.0.47            0c-54-15-a9-6e-65     dynamique
  10.33.0.110           94-b8-6d-59-34-45     dynamique
  10.33.0.196           f8-59-71-83-5e-78     dynamique
  10.33.0.234           70-c9-4e-90-0a-c7     dynamique
  10.33.1.18            2c-20-0b-41-55-95     dynamique
  10.33.1.27            24-18-1d-ae-9a-2d     dynamique
  10.33.1.48            30-24-32-5a-6f-bd     dynamique
  10.33.1.68            d8-bb-2c-b5-88-b6     dynamique
  10.33.1.80            40-9f-38-1c-a9-db     dynamique
  10.33.1.83            ca-ff-a4-e7-e1-a0     dynamique
  10.33.1.91            b4-69-21-47-b9-ea     dynamique
  10.33.1.135           6c-e8-5c-3d-f4-85     dynamique
  10.33.1.147           b0-70-2d-b8-c2-c3     dynamique
  10.33.1.149           dc-a2-66-21-d4-7d     dynamique
  10.33.1.185           04-d6-aa-72-7b-34     dynamique
  10.33.1.245           e4-0e-ee-73-78-67     dynamique
  10.33.2.0             34-f6-4b-07-51-cf     dynamique
  10.33.2.4             b8-8a-60-5b-d9-98     dynamique
  10.33.2.36            40-a3-cc-b0-90-d2     dynamique
  10.33.2.86            70-c9-4e-be-d4-b5     dynamique
  10.33.2.87            88-b1-11-5a-63-33     dynamique
  10.33.2.100           9c-30-5b-e0-f7-d3     dynamique
  10.33.2.107           68-ec-c5-e6-19-be     dynamique
  10.33.2.119           a8-5c-2c-61-13-57     dynamique
  10.33.2.138           3c-f8-62-9b-4f-5b     dynamique
  10.33.2.186           28-c6-3f-9e-d7-89     dynamique
  10.33.2.196           98-22-ef-58-bc-87     dynamique
  10.33.2.205           dc-2b-2a-50-1c-be     dynamique
  10.33.2.243           d4-6d-6d-9b-f9-d8     dynamique
  10.33.2.244           1c-4d-70-dd-e8-bb     dynamique
  10.33.3.28            a0-56-f3-9f-e0-f1     dynamique
  10.33.3.42            f4-8c-50-ab-c6-06     dynamique
  10.33.3.65            18-5e-0f-d9-f5-3f     dynamique
  10.33.3.121           04-d3-b0-1e-c4-36     dynamique
  10.33.3.126           8c-85-90-ab-b5-84     dynamique
  10.33.3.172           9c-b6-d0-20-69-a9     dynamique
  10.33.3.236           88-78-73-97-91-6d     dynamique
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  10.33.3.254           94-0c-6d-84-50-c8     dynamique
  10.33.3.255           ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

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

Machine | `net1` | `net2` | Adresse MAX
--- | --- | --- | ---
`client1.tp4` | `10.1.0.10` | X | `0a:00:27:00:00:46`
`router1.tp4` | `10.1.0.254` | `10.2.0.254` | `08:00:27:9c:42:37`
`server1.tp4` | X | `10.2.0.10` | `0a:00:27:00:00:4b`