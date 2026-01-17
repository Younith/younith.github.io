---
title: Part 3
parent: Week 9
layout: default
nav_order: 3
---
# Completion

{: .important-title }
> Test 1 Success
>
> ![](/assets/images/CCNA 3/Week-9/Test-1-Success.png)

{: .important-title }
> Test 3 Success
>
> ![](/assets/images/CCNA 3/Week-9/Test-3-Success.png)

{: .important-title }
> Test 4 Success
>
> ![](/assets/images/CCNA 3/Week-9/Test-4-Success.png)

{: .important-title }
> Ping PC-A to PC-B
>
> ![](/assets/images/CCNA 3/Week-9/Ping_PC-A_PC-B.png)

{: .important-title }
> TFTP Server
> 
> ![](/assets/images/CCNA 3/Week-9/TFTP_Server.png)

# Device Configuration
## Full Topology 

{: .important-title }
> Full Topology
>
> ![](/assets/images/CCNA 3/Week-9/Week-9-P3-Full-Topology.png)

## LEFT
### R1

```
enable
config terminal

no ip domain lookup 

hostname R1

ip domain-name ccna-lab.com

enable secret ciscoenpass

line console 0
password ciscoconpass
login
exit 

security passwords min-length 10

username admin secret admin1pass

line vty 0 15
login local
transport input ssh 
exit

service password-encryption 

banner motd #Unauthorized Access is Prohibited#

interface g0/0/0
description Connect to R2
ip address 10.67.254.2 255.255.255.252
no shutdown 

interface g0/0/1
description Connect to LAN A
ip address 192.168.1.1 255.255.255.0
no shutdown

interface loopback 0
ip address 10.52.0.1 255.255.255.248
exit

crypto key generate rsa 
How many bits in the modulus [512]: 1024

#### Part 2: Configure Single Area OSPFv2

configure terminal 

router ospf 1
router-id 0.0.0.1
network 10.67.254.0 0.0.0.3 area 0
network 192.168.1.0 0.0.0.255 area 0
network 10.52.0.0 0.0.0.7 area 0

#### Part 3: Optimize Single-Area OSPFv2

end
configure terminal 

router ospf 1

passive-interface g0/0/1
passive-interface loopback 0

auto-cost reference-bandwidth 1000
exit

interface loopback 0
ip ospf network point-to-point 
exit

interface g0/0/0
ip ospf hello-interval 30

#### Part 4: Configure Access Control, NAT, and perform configuration backup

router ospf 1
no network 192.168.1.0 0.0.0.255 area 0
exit

access-list 1 permit 192.168.1.0 0.0.0.255

ip nat inside source list 1 interface g0/0/0 overload 

interface g0/0/0
ip nat outside
interface g0/0/1
ip nat inside 

copy running-config tftp
Address or name of remote host []? 10.67.1.50
```

### S1

```
enable
configure terminal

no ip domain-lookup 

hostname S1

ip domain-name ccna-lab.com

enable secret ciscoenpass

line console 0
password ciscoconpass
login
exit

interface range f0/1-4, f0/7-24, g0/1-2
shutdown 
exit

username admin secret admin1pass

line vty 0 15
login local 
transport input ssh 
exit

service password-encryption 

banner motd #Unauthorized access or use prohibited#

crypto key generate rsa 
How many bits in the modulus [512]: 1024

interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown 
exit

ip default-gateway 192.168.1.1
end

#### Part 4: Configure Access Control, NAT, and perform configuration backup

copy running-config tftp
Address or name of remote host []? 10.67.1.50
```

### PC-A

```
- Address 		= 192.168.1.50
- Subnet Mask 	= 255.255.255.0
- Gateway 		= 192.168.1.1
- DNS 			= 
```

## RIGHT
### R2

```
enable
config terminal

no ip domain lookup 

hostname R2

ip domain-name ccna-lab.com

enable secret ciscoenpass

line console 0
password ciscoconpass
login
exit 

security passwords min-length 10

username admin secret admin1pass

line vty 0 15
login local
transport input ssh 
exit

service password-encryption 

banner motd #Unauthorized Access is Prohibited#

interface g0/0/0
description Connect to R1
ip address 10.67.254.1 255.255.255.252
no shutdown 

interface g0/0/1
description Connect to LAN B
ip address 10.67.1.1 255.255.255.0
no shutdown

interface loopback 0
ip address 209.165.201.1 255.255.255.224
exit

crypto key generate rsa
How many bits in the modulus [512]: 1024

#### Part 2: Configure Single Area OSPFv2

configure terminal 

router ospf 1
router-id 0.0.0.2
network 10.67.254.0 0.0.0.3 area 0
network 10.67.1.0 0.0.0.255 area 0

#### Part 3: Optimize Single-Area OSPFv2

end
configure terminal 

router ospf 1

passive-interface g0/0/1
passive-interface loopback 0

auto-cost reference-bandwidth 1000

ip route 0.0.0.0 0.0.0.0 loopback 0
router ospf 1
default-information originate 
exit 

interface g0/0/0
ip ospf hello-interval 30

ip ospf priority 50
exit

#### Part 4: Configure Access Control, NAT, and perform configuration backup

ip access-list extended R2-SECURITY
deny tcp any host 209.165.201.1 eq 443
deny tcp any host 209.165.201.1 eq 22
permit ip any any 
exit

interface g0/0/0
ip access-group R2-SECURITY in

copy running-config tftp
Address or name of remote host []? 10.67.1.50
```

### S2

```
enable
configure terminal 

no ip domain-lookup 

hostname S2

ip domain-name ccna-lab.com

enable secret ciscoenpass

line console 0
password ciscoconpass
login
exit

interface range f0/1-4, f0/6-17, f0/19-24, g0/1-2
shutdown 
exit

username admin secret admin1pass

line vty 0 15
login local 
transport input ssh 
exit

service password-encryption 

banner motd #Unauthorized access or use prohibited#

crypto key generate rsa 
How many bits in the modulus [512]: 1024

interface vlan 1
ip address 10.67.1.2 255.255.255.0
no shutdown 
exit

ip default-gateway 10.67.1.1
end

#### Part 4: Configure Access Control, NAT, and perform configuration backup

copy running-config tftp
Address or name of remote host []? 10.67.1.50
```

### PC-B

```
- Address 		= 10.67.1.50
- Subnet Mask 	= 255.0.0.0
- Gateway 		= 10.67.1.1
- DNS 			= 
```
