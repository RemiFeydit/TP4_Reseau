# TP 4 - Spéléologie réseau : descente dans les couches

# I. Mise en place du Lab

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
# Mise en place du routage statique

`ip route show` :
```bash
[remi@router1 ~]$ ip route show
10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.254 metric 100
10.2.0.0/24 dev enp0s9 proto kernel scope link src 10.2.0.254 metric 101
```
