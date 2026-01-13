---
title: Week 6
parent: Firewall Security
layout: default
nav_order: 6
---
# Completion

{: .important-title }
> Both side can reach every webserver
>
> ![](/assets/images/Firewall Security/Week-6/Left-Right-reach-webservers.png)

# Device Configuration
## Full Topology

{: .important-title }
> Full Topology
>
> ![](/assets/images/Firewall Security/Week-6/Full-Topology.png)

## Middle
### ISP

```
/ip address add address=1.1.1.254/24 interface=ether1
/ip address add address=2.2.2.254/24 interface=ether2
/ip address add address=8.8.8.1/24 interface=ether3


/ip firewall address-list
add list="BOGONS" address=0.0.0.0/8
add list="BOGONS" address=10.0.0.0/8
add list="BOGONS" address=100.64.0.0/10
add list="BOGONS" address=127.0.0.0/8
add list="BOGONS" address=169.254.0.0/16
add list="BOGONS" address=172.16.0.0/12
add list="BOGONS" address=192.0.0.0/24
add list="BOGONS" address=192.0.2.0/24
add list="BOGONS" address=192.168.0.0/16
add list="BOGONS" address=198.18.0.0/15
add list="BOGONS" address=198.51.100.0/24
add list="BOGONS" address=203.0.113.0/24
add list="BOGONS" address=224.0.0.0/3

/ip firewall filter
add action=accept chain=forward connection-state=established,related

add action=drop chain=forward comment="Block Bogon IP Addresses" in-interface=ether1 src-address-list=BOGONS

add action=drop chain=forward comment="Block Bogon IP Addresses" in-interface=ether2 src-address-list=BOGONS
```

### ISP-DNS

```
enable
conf t
interface FastEthernet 0/0
ip address 8.8.8.8 255.255.255.0
no shut
exit

ip dns server
ip domain-lookup
ip domain name lab
ip host www.beltei.edu 2.2.2.2 
ip host app.beltei.edu 2.2.2.2
ip host www.beltei.com 1.1.1.1
ip host app.beltei.com 1.1.1.1

ip route 0.0.0.0 0.0.0.0 8.8.8.1
exit
```
## Left Side
### Mikrotik
#### Firewall Checklist

- [x] DNS Servers can only respond to `clients` and talk to `external-dns-server`
	- [x] can't ping `clients` & can resolve `webservers` & not ping `webservers`
- [x] Clients can only talk to `webservers` and `internal-dns-server`
	- [x] can talk to `webservers` & can resolve `webservers` & can't ping `external-dns-server`
- [x] Web Servers can only respond to `clients`
	- [x] can't ping `anyone`

#### Router Configuration

```
/ip address add address=192.168.2.254/24 interface=ether2
/ip address add address=192.168.1.254/24 interface=ether3
/ip address add address=192.168.3.254/24 interface=ether4
/ip address add address=1.1.1.1/24 interface=ether1

/ip route add dst-address=2.2.2.2/32 gateway=1.1.1.254
/ip route add dst-address=8.8.8.8/32 gateway=1.1.1.254

/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

/ip firewall nat add chain=dstnat action=dst-nat dst-address=1.1.1.1 dst-port=80 to-addresses=192.168.2.1 protocol=tcp
```

#### Address-list Configuration

```
/ip firewall address-list
add list=web-servers address=192.168.2.1/32
add list=web-servers address=2.2.2.2
add list=web-clients address=192.168.3.1/32
add list=web-clients address=2.2.2.2

add list=server-internal-dns address=192.168.1.1/32
add list=clients-internal-dns address=192.168.3.1/32

add list=server-external-dns address=8.8.8.8/32
```

#### Firewall Configuration

```
/ip firewall filter
add action=accept chain=forward connection-state=established,related

add action=accept chain=forward dst-address-list=server-external-dns src-address-list=server-internal-dns protocol=udp dst-port=53 comment="Allow Internal DNS server -> External DNS server"

add action=accept chain=forward dst-address-list=web-servers src-address-list=web-clients protocol=tcp dst-port=80 comment="Allow Client and 2.2.2.2 (http)-> web-servers"

add action=accept chain=forward dst-address-list=server-internal-dns src-address-list=clients-internal-dns protocol=udp dst-port=53 comment="Allow Clients (dns)-> Internal DNS server"

add action=drop chain=forward comment="Drop Forward"

add action=accept chain=input connection-state=established,related

add action=accept chain=input protocol=tcp dst-port=23 comment="Allow Telnet Traffic"

add action=drop chain=input comment="Drop Input"
```
### R1

```
enable
conf t
interface FastEthernet 0/0
ip address 192.168.1.1 255.255.255.0
no shut
exit

ip dns server
ip domain-lookup
ip domain name lab
ip host www.beltei.com 192.168.2.1
ip host app.beltei.com 192.168.2.1
ip name-server 8.8.8.8

ip route 0.0.0.0 0.0.0.0 192.168.1.254
exit
```

### Windows Server 1

```
1. Installed IIS role
2. created website = www.beltei.com in C:/inetpub/wwwroot/
	- Type : http
	- Host Name : www.beltei.com
	- Ip address : *
3. created website = app.beltei.com in C:/inetpub/app.beltei.com/
	- Type : http
	- Host Name : app.beltei.com
	- Ip address : *

IP configuration:
	- Address = 192.168.2.1
	- Subnet Mask = 255.255.255.0
	- Gateway = 192.168.2.254
	- DNS = 192.168.1.1
```

### Ubuntu Desktop 2

```
Wired Connection > Wired Settings > *gear icon* > Ipv4 > IPv4 Method > Manual :
	- Address = 192.168.3.1
	- Subnet Mask = 255.255.255.0
	- Gateway = 192.168.3.254
	- DNS = 192.168.1.1
```

## Right Side

### Mikrotik 2

#### Firewall Checklist

- [x] DNS Servers can only respond to `clients` and talk to `external-dns-server`
	- [x] can't ping `clients` & can resolve `webservers` & not ping `webservers`
- [x] Clients can only talk to `webservers` and `internal-dns-server`
	- [x] can talk to `webservers` & can resolve `webservers` & can't ping `external-dns-server`
- [x] Web Servers can only respond to `clients`
	- [x] can't ping `anyone`
	
#### Router Configuration

```
/ip address add address=2.2.2.2/24 interface=ether1
/ip address add address=192.168.1.254/24 interface=ether3
/ip address add address=192.168.2.254/24 interface=ether2
/ip address add address=192.168.4.254/24 interface=ether4

/ip route add dst-address=1.1.1.1/32 gateway=2.2.2.254
/ip route add dst-address=8.8.8.8/32 gateway=2.2.2.254

/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

/ip firewall nat add chain=dstnat action=dst-nat dst-address=2.2.2.2 dst-port=80 to-addresses=192.168.2.1 protocol=tcp

```

#### Address-list Configuration

```
/ip firewall address-list
add list=web-servers address=192.168.2.1/32
add list=web-servers address=1.1.1.1
add list=web-clients address=192.168.4.1/32
add list=web-clients address=1.1.1.1

add list=server-internal-dns address=192.168.1.1/32
add list=clients-internal-dns address=192.168.4.1/32

add list=server-external-dns address=8.8.8.8/32
```
#### Firewall Configuration

```
/ip firewall filter
add action=accept chain=forward connection-state=established,related

add action=accept chain=forward dst-address-list=server-external-dns src-address-list=server-internal-dns protocol=udp dst-port=53 comment="Allow Internal DNS server -> External DNS server"

add action=accept chain=forward dst-address-list=web-servers src-address-list=web-clients protocol=tcp dst-port=80 comment="Allow Client and 2.2.2.2 (http)-> web-servers"

add action=accept chain=forward dst-address-list=server-internal-dns src-address-list=clients-internal-dns protocol=udp dst-port=53 comment="Allow Clients (dns)-> Internal DNS server"

add action=drop chain=forward comment="Drop Forward"

add action=accept chain=input connection-state=established,related

add action=accept chain=input protocol=tcp dst-port=23 comment="Allow Telnet Traffic"

add action=drop chain=input comment="Drop Input"

```
### DNS

```
enable
conf t
interface FastEthernet 0/0
ip address 192.168.1.1 255.255.255.0
no shut
exit

ip dns server
ip domain-lookup
ip domain name lab
ip host www.beltei.edu 192.168.2.1
ip host app.beltei.edu 192.168.2.1
ip name-server 8.8.8.8

ip route 0.0.0.0 0.0.0.0 192.168.1.254
exit
```

### Windows Server 1

```
1. Installed IIS role
2. created website = www.beltei.edu in C:/inetpub/wwwroot/
	- Type : http
	- Host Name : www.beltei.edu
	- Ip address : *
3. created website = app.beltei.edu in C:/inetpub/app.beltei.com/
	- Type : http
	- Host Name : app.beltei.edu
	- Ip address : *

IP configuration:
	- Address = 192.168.2.1
	- Subnet Mask = 255.255.255.0
	- Gateway = 192.168.2.254
	- DNS = 192.168.1.1
```

### Ubuntu Desktop 1

```
Wired Connection > Wired Settings > *gear icon* > Ipv4 > IPv4 Method > Manual :
	- Address = 192.168.4.1
	- Subnet Mask = 255.255.255.0
	- Gateway = 192.168.4.254
	- DNS = 192.168.1.1
```