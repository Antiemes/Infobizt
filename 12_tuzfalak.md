# Tűzfalak

## Netfilter tűzfal

A Netfilter a Linux kernel beépített tűzfala. Alapesetben statikus csomagszűrő
tűzfal, de számos kiegészítéssel rendelkezik, amik ezt a funkcionalitást
kibővítik.

A Netfilter szolgáltatásaihoz az aktuális paracssori eszköz az `nftables`, vagy `nft`, az `iptables` paranccsal érhetőek el. Az újabb rendszereken
az `nftables` az alapértelmezett, így az `iptables`
ezt kell telepítenünk (a minimál telepítésnek nem része).

```bash
apt-get install iptables
```

Az aktuális szabályok listázása:

```bash
iptables -S
```

```bash
iptables -L -n -v
```

```bash
iptables -t mangle -L -n -v
```



táblák
filter szűrés
nat NAT
mangle IP header módosítás
RAW connection tracking
security SElinux

Chains

prerouting
Csomag belépésekor, routing döntés előtt

input
Network stack belépéskor

forward
átmenő forgalom

output
A rendszerben keletkező csomagok

postrouting
routing decision után


filter: input, forward, output
mangle: mindegyik
DNAT: prerouting, output
SNAT: input, postrouting


Bejövő csomagok: prerouting -> input
Bejövő csomagok, amik másfele mennek tovább: prerouting -> forward -> postrouting
Helyileg generált csomagok: output -> postrouting

Rules

Ahogy egy csomag végigmegy az egyes chain-eken, a szabályokra próbál match-elni, sorrendben. Ha nem igaz a szabály a csomagra, akkor megy tovább a következőre. 

Minden szabálynak van két része: matching, target.
Match: Protokoll, dst, src, dport, sport, input iface, output iface, header-ek, stb.

Target:
  terminating, non-terminating
  terminating: accept, drop, queue, reject, return, vagy átmozgatás egy felhasználó által definiált chain-be.

  Non-term: Folytatja a chain-t.

iptables -A INPUT -s a.b.c.d -j DROP

Deny-everything policy
Accept-everything policy

SYN flood

Ping flood
A source address hamis

```bash
iptables --flush
iptables -F
```

-A --append
-P --policy

```bash
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
iptables --policy INPUT DROP
iptables --policy OUTPUT DROP
iptables --policy FORWARD DROP
```

Target
  ACCEPT
  DROP nem kuld vissza semmit
  REJECT icmp-port-unreacheable

$SERVER_IP

/etc/systemd/system/iptables-persistent.service

[Unit] 
Description=runs iptables restore on boot
ConditionFileIsExecutable=/etc/iptables/restore-iptables.sh
After=network.target[Service]
Type=forking
ExecStart=/etc/iptables/restore-iptables.sh
start TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no[Install]
WantedBy=multi-user.target

## Firewalld

https://blog.myhro.info/2021/12/configuring-firewalld-on-debian-bullseye

## ufw

# Port knocking

8881
7777
9991

nmap -Pn --host-timeout 201 --max-retries 0  -p 1111 host

/lib/modules
netfilter
/proc/net/xt_recent

iptables -A INPUT -p icmp --icmp-type echo-request -j LOG



nmap -Pn --host-timeout 100 --max-retries 0 -p 1111 192.168.8.55
#nmap -Pn --host-timeout 100 --max-retries 0 -p 2222 192.168.8.55
#nmap -Pn --host-timeout 100 --max-retries 0 -p 3333 192.168.8.55


#nmap -Pn --host-timeout 100 --max-retries 0 -p 8888 192.168.8.55

sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

