---
title: Week 12
parent: Firewall Security
layout: default
nav_order: 12
---
# Completion
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .important-title }
> `Client-1` can reach both the internet through the proxy
> 
> ![](/assets/images/Firewall Security/Week-12/Accessing-Reddit.png)

# Device Configuration
## Full Topology

{: .important-title }
> Full Topology
> 
> ![](/assets/images/Firewall Security/Week-12/Full-topology.png)

## DNS-1

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

ip name-server 1.1.1.1
ip name-server 8.8.8.8

ip route 0.0.0.0 0.0.0.0 192.168.1.254
exit
write
```

## Proxy

```
#/etc/squid/squid.conf

acl All src all
acl manager proto cache_object
acl CONNECT method CONNECT
acl MyNetwork src 192.168.0.0/16

http_access allow manager localhost
http_access deny manager 
http_access deny !Safe_ports 
http_access deny CONNECT !SSL_ports
http_access allow MyNetwork

http_access deny all

```

## Client-1

```
#
# This is a sample network config, please uncomment lines to configure the network
#
# Uncomment this line to load custom interface files
# source /etc/network/interfaces.d/*
# Static config for eth0

auto eth0
iface eth0 inet static
address 192.168.3.1
netmask 255.255.255.0
gateway 192.168.3.254
up echo nameserver 192.168.1.1 > /etc/resolv.conf

# DHCP config for eth0
#auto eth0
#iface eth0 inet dhcp
# hostname webterm-1
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

edit port2
set role LAN
set alias PROXY
set ip 192.168.2.254 255.255.255.0
set allowaccess ping
set mode static
next

edit port3
set role LAN
set alias LOCAL_DNS
set ip 192.168.1.254 255.255.255.0
set allowaccess ping
set mode static
next

edit port4
set role LAN
set alias CLIENT1
set ip 192.168.3.254 255.255.255.0
set allowaccess ping ssh http https
set mode static
next
end

config firewall policy (WIP)
    edit 1
		set name "PROXY_LOCAL_DNS"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "DNS"
    next
    edit 2
		set name "PROXY_WAN"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set nat enable
        set ippool disable
    next
    edit 3
		set name "CLIENT1_PROXY"
        set srcintf "port4"
        set dstintf "port2"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS" "SQUID"
    next
    edit 4
		set name "LOCAL_DNS_WAN"
        set srcintf "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "DNS"
        set nat enable
        set ippool disable
    next
end
```