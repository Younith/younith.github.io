---
title: Part 2
parent: Week 9
layout: default
nav_order: 2
---

# Completion

{: .important-title }
> **SV1**  & **SV2** can ping eachother
>
> ![](/assets/images/CCNA 3/Week-9/LEFT-&-RIGHT-Ping-Eachother.png)

# Device Configuration
## Full Topology

{: .important-title }
> Full Topology
>
> ![](/assets/images/CCNA 3/Week-9/Full-Topology.png)

## LEFT
### R1

```
enable
conf t

hostname R1
interface loopback 1
ip address 1.1.1.1 255.255.255.0

interface serial 0/0/0
ip address 172.16.1.1 255.255.255.0
no shut

interface gigabitEthernet 0/0
ip address 192.18.10.254 255.255.255.0
no shut

exit
router ospf 1
network 192.18.10.0 0.0.0.255 area 0
network 172.16.1.0 0.0.0.255 area 0

router-id 1.1.1.1
end
```

### SV1

```
IP configuration:
	- Address     = 192.18.10.1
	- Subnet Mask = 255.255.255.0
	- Gateway     = 192.18.10.254
```

## RIGHT
### R2

```
enable
conf t

hostname R2
interface loopback 1
ip address 2.2.2.2 255.255.255.0

interface serial 0/0/0
ip address 172.16.1.2 255.255.255.0
no shut


interface gigabitEthernet 0/0
ip address 192.17.10.254 255.255.255.0
no shut

exit
router ospf 1
network 192.17.10.0 0.0.0.255 area 0
network 172.16.1.0 0.0.0.255 area 0

router-id 2.2.2.2
end
```

### SV2

```
IP configuration:
	- Address     = 192.17.10.1
	- Subnet Mask = 255.255.255.0
	- Gateway     = 192.17.10.254
```
