---
title: Week 10
parent: Firewall Security
layout: default
nav_order: 10
---

# Completion

{: .important-title }
> `Client1` can reach both webservers
> 
> ![](/assets/images/Firewall Security/Week-10/Client1-both-webserver.png)

{: .important-title }
> `Client2` can reach both webservers
> 
> ![](/assets/images/Firewall Security/Week-10/Client2-both-webserver.png)

# Device Configuration
## Full Topology

{: .important-title }
> Full Topology
> 
> ![](/assets/images/Firewall Security/Week-10/Full-Topology.png)

## fortigate

```
config system interface

edit port1
set alias WAN
set mode dhcp  
set defaultgw enable
set dns-server-override enable
next  

edit port2
set role LAN
set alias LOCAL_DNS
set ip 192.168.1.254 255.255.255.0
set allowaccess ping
set mode static
next

edit port3
set role LAN
set alias LOCAL_WEB
set ip 192.168.6.254 255.255.255.0
set allowaccess ping
set mode static
next

edit port4
set role LAN
set alias LOCAL_CLIENT
set ip 192.168.5.254 255.255.255.0
set allowaccess ping
set mode static
next
end

*DHCP on gui for port4*

config firewall vip
    edit "Internal_WebServer"
		set extip 0.0.0.0
        set extintf "port1"
        set mappedip "192.168.6.1"
		set mappedport 80
		set extport 80
		set portforward enable
    next
end

config firewall policy
    edit 1
		set name "LOCAL_CLIENT_LOCAL_DNS"
        set srcintf "port4"
        set dstintf "port2"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "dns"
    next
    edit 2
		set name "LOCAL_CLIENT_LOCAL_WEB"
        set srcintf "port4"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "http"
    next
    edit 3
        set name "LAN3_WAN"
        set srcintf "port4"
        set dstintf "port1"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "HTTP" "HTTPS" "PING"
        set nat enable
        set ippool disable
    next
    edit 4
		set name "LOCAL_DNS_GLOBAL_DNS"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "DNS"
        set nat enable
    next
    edit 8
        set name "WAN_WebServer-http"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
		set dstaddr Internal_WebServer
        set action accept
        set schedule "always"
        set service "HTTP"
        set nat enable
    next
end
end
end

```

### DNS-1

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
ip host www.beltei.edu 192.168.6.1

ip name-server 8.8.8.8

ip route 0.0.0.0 0.0.0.0 192.168.1.254
exit
```

### Beltei.com

```
Webserver (nginx):
	- www.beltei.com
		- index.html > This is beltei.com
		
IP (netplan):
	ip 		: 192.168.3.1/24
	dfgw	: 192.168.3.254
	dns		: 192.168.2.1
```

### Client1

```
Wired Connection > Wired Settings > *gear icon* > Ipv4 > IPv4 Method > Manual :
	- Address 		= 192.168.5.1
	- Subnet Mask 	= 255.255.255.0
	- Gateway 		= 192.168.5.254
	- DNS 			= 192.168.1.1
```

## Mikrotik

```
/ip dhcp-client add interface=ether1 disabled=no use-peer-dns=yes use-peer-ntp=yes add-default-route=yes

/ip address add address=192.168.2.254/24 interface=ether2
/ip address add address=192.168.3.254/24 interface=ether3
/ip address add address=192.168.4.254/24 interface=ether4

*GUI DHCP config on ether4*

/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

/ip firewall nat add chain=dstnat action=dst-nat in-interface=ether1 dst-port=80 to-addresses=192.168.3.1 to-port=80 protocol=tcp


/ip firewall address-list
add list=server-internal-dns address=192.168.2.1/32
add list=server-external-dns address=192.168.122.10/32

/ip firewall filter
add action=accept chain=forward connection-state=established,related

add action=accept chain=forward dst-address-list=server-external-dns src-address-list=server-internal-dns protocol=udp dst-port=53 comment="ALLOW : server-internal-dns > server-external-dns (udp/53)"

add action=accept chain=forward protocol=tcp dst-port=80 comment="ALLOW : * > web-servers (tcp/80)"

add action=accept chain=forward dst-address-list=server-internal-dns protocol=udp dst-port=53 comment="ALLOW : * > server-internal-dns (udp/53)"

add action=drop chain=forward comment="Drop Forward"
```

### DNS-3

```
enable
conf t
interface FastEthernet 0/0
ip address 192.168.2.1 255.255.255.0
no shut
exit

ip dns server
ip domain-lookup
ip domain name lab
ip host www.beltei.com 192.168.3.1

ip name-server 192.168.122.10

ip route 0.0.0.0 0.0.0.0 192.168.2.254
exit
```

### Beltei.edu

```
Webserver (nginx):
	- www.beltei.edu
		- index.html > This is beltei.edu
		
IP (netplan):
	ip 		: 192.168.6.1/24
	dfgw	: 192.168.6.254
	dns		: 192.168.1.1

```

### Client2

```
Wired Connection > Wired Settings > *gear icon* > Ipv4 > IPv4 Method > Manual :
	- Address 		= 192.168.4.1
	- Subnet Mask 	= 255.255.255.0
	- Gateway 		= 192.168.4.254
	- DNS 			= 192.168.2.1
```

## DNS-Global

```
enable
conf t
interface FastEthernet 0/0
ip address 192.168.122.10 255.255.255.0
no shut
exit

ip dns server
ip domain-lookup
ip domain name lab
ip host www.beltei.edu 192.168.122.144
ip host www.beltei.com 192.168.122.39

exit
```