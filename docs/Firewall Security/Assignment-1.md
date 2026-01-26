---
title: Assignment 1
parent: Firewall Security
layout: default
nav_order: 1
---

# Completion

{: .important-title }
> `Client1` can reach the webserver & the internet
> 
> ![](/assets/images/Firewall Security/Assignment-1/Client 1-to-internet-and-webserver.png)

{: .important-title }
> Mikrotik's route-table 
> 
> ![](/assets/images/Firewall Security/Assignment-1/mikrotik-route-table.png)

{: .important-title }
> Fortigate's route-table 
> 
> ![](/assets/images/Firewall Security/Assignment-1/fortigate-route-table.png)

{: .important-title }
> Cisco's route-table 
> 
> ![](/assets/images/Firewall Security/Assignment-1/cisco-route-table.png)

# Device Configuration
## Full Topology

{: .important-title }
> Full Topology
> 
> ![](/assets/images/Firewall Security/Assignment-1/Full-Topology.png)

## Mikrotik

```
/ip dhcp-client add interface=ether1 disabled=no use-peer-dns=yes use-peer-ntp=yes add-default-route=yes

/ip address add address=192.168.1.254/24 interface=ether3
/ip address add address=192.168.2.254/24 interface=ether4

/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

# Mikrotik > Fortigate (10.10.10.0/30)
/ip address add address=10.10.10.1/30 interface=ether2

/routing rip instance
add name=rip-v2 originate-default=never redistribute=connected

/routing rip interface-template
add instance=rip-v2 interfaces=ether2 disabled=no

/ip pool add name=dhcp_pool4 ranges=192.168.2.1-192.168.2.253
*dhcp server set-up via gui*

/ip firewall address-list
add list=internet_clients address=192.168.2.0/24
add list=dns_global_clients address=192.168.1.1

/ip firewall filter
add action=accept chain=forward connection-state=established,related

add action=accept chain=forward protocol=udp dst-port=53 dst-address=192.168.1.1 comment="ALLOW : * > DNS"

add action=accept chain=forward protocol=tcp dst-port=80 dst-address=192.168.5.1 comment="ALLOW : internet_clients > Webserver (tcp/80)"

add action=accept chain=forward src-address-list=dns_global_clients protocol=udp dst-port=53 out-interface=ether1 comment="ALLOW : dns_global_clients > internet (udp/53)"

add action=accept chain=forward src-address-list=internet_clients protocol=tcp dst-port=80,443 out-interface=ether1 comment="ALLOW : internet_clients > internet (tcp/80,443)" 

add action=drop chain=forward comment="Drop Forward"
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
ip host www.beltei.com 192.168.5.1
ip host app.beltei.com 192.168.5.1
ip name-server 8.8.8.8

ip route 0.0.0.0 0.0.0.0 192.168.1.254
exit
```

### Client1

```
IP Config:
	ip (DHCP)	: 192.168.2.* 
	dfgw		: 192.168.2.254
	dns			: 192.168.1.1
```

## Fortigate

```
config system interface  
	edit port1
		set alias WAN
		set mode dhcp  
		set defaultgw enable
		set dns-server-override enable
	next  
	
	# Mikrotik < Fortigate (10.10.10.0/30)
	
	edit port2
		set role LAN
		set alias mikrotik
		set ip 10.10.10.2 255.255.255.252
		set allowaccess ping 
		set mode static
	next
	
	# Fortigate > Cisco (10.10.10.4/30)
	
	edit port3
		set role LAN
		set alias cisco
		set ip 10.10.10.5 255.255.255.252
		set allowaccess ping 
		set mode static
	next
	
	edit port4
		set role LAN
		set alias LAN3
		set ip 192.168.3.254 255.255.255.0
		set allowaccess ping 
		set mode static
	next
	**dhcp server configured on this interface through gui**
	end

## Rip-v2

config router rip
	config network
		edit 1
			set prefix 10.10.10.0 255.255.255.252
		next
		
		edit 2
			set prefix 10.10.10.4 255.255.255.252
		next
		
		edit 3
			set prefix 192.168.3.0 255.255.255.0
		next
		end
	end

config interface
	edit port2
		set receive-version 2
		set send-version 2
	next
	edit port3
		set receive-version 2
		set send-version 2
	next
	end

config firewall policy
    edit 1
		set name "mikrotik_cisco"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 2
		set name "cisco_mikrotik"
        set srcintf "port3"
        set dstintf "port2"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 3
        set name "LAN3_WAN"
        set srcintf "port4"
        set dstintf "port1"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service http https ping
        set nat enable
        set ippool disable
    next
    edit 4
        set name "LAN3_mikrotik"
        set srcintf "port4"
        set dstintf "port2"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service dns ping
    next
    edit 5
        set name "LAN3_cisco"
        set srcintf "port4"
        set dstintf "port3"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service http ping
    next
end

```


### Client3

```
IP Config:
	ip (DHCP)	: 192.168.3.* 
	dfgw		: 192.168.3.254
	dns			: 192.168.1.1
```

## Cisco

```
enable

conf t
interface f0/0
ip address dhcp
ip nat outside
## these don't seems to do anything
ip virtual-reassembly
ip route-cache cef
ip route-cache
ip mroute-cache
ip access-group 102 out
no shut

interface f1/0
ip add 192.168.6.254 255.255.255.0
ip nat inside
ip access-group 101 in
no shut

interface f1/0.10
encapsulation dot1q 10
ip add 192.168.5.254 255.255.255.0
ip nat inside

# Fortigate < Cisco (10.10.10.4/30)
interface f0/1
ip add 10.10.10.6 255.255.255.252
ip nat inside
no shut
exit

router rip
version 2
no auto-summary
network 10.10.10.4
network 192.168.6.0
network 192.168.5.0
exit


## DHCP

ip dhcp pool lan6
network 192.168.6.0 255.255.255.0
dns-server 192.168.1.1
default-router 192.168.6.254

## NAT

access-list 1 permit 192.168.6.0 0.0.0.255
ip nat inside source list 1 interface f0/0 overload

## ACLs

ip access-list extended 101
permit tcp any 192.168.6.0 0.0.0.255 established
permit tcp 192.168.6.0 0.0.0.255 any eq 443
permit tcp 192.168.6.0 0.0.0.255 any eq 80
permit udp 192.168.6.0 0.0.0.255 host 192.168.1.1 eq 53
deny tcp any any
deny udp any any

ip access-list extended 102
permit tcp any any eq 443
permit tcp any any eq 80
deny ip any any
```

### SW 2

```
enable
conf t

hostname SW2

vlan 10
name WEB
exit

interface gi0/1
switchport mode access
switchport access vlan 10

interface gi0/0
switchport trunk encapsulation dot1q
switchport mode trunk
exit
```

### Webserver

```
Webserver (nginx):
	- www.beltei.com
		- index.html > <h1>This is www.beltei.com</h1>
		
	- app.beltei.com
		- index.html > <h1>This is app.beltei.com</h1>

IP (netplan):
	ip 		: 192.168.5.1/24
	dfgw	: 192.168.5.254
	dns		: 192.168.1.1
```

### Client2

```
IP Config:
	ip (DHCP)	: 192.168.6.* 
	dfgw		: 192.168.6.254
	dns			: 192.168.1.1
```