# Cisco_ios_cheatsheet
Cisco CLI reference


## Quick Navigtation
+ [Setup](#setup)
    - [Intialize](#intialize)
    - [Set Clock](#set-clock)
+ [Interfaces](#interfaces)
    - [Assign Static IP to Interface](#assign-static-ip-to-interface)
    - [Interface Ranges](#interface-ranges)
+ [Interface Verification](#interface-verification)
    - [Remove IP Addresses](#remove-ip-addresses)
+ [DHCP](#dhcp)
    - [Enable Router DHCP Server](#enable-router-dhcp-server)
    - [DHCP Verification](#dhcp-verification)
    - [Disable DHCP](#disable-dhcp)
    - [Re-enabled DHCP](#re-enabled-dhcp)
    - [Create VLAN DHCP](#create-vlan-dhcp)
    - [Verify DHCP Pool](#verify-dhcp-pool)
    - [Delete DHCP Pool](#delete-dhcp-pool)
+ [VLANs](#vlans)
    - [VLAN Creation](#vlan-creation)
    - [Port Assignment](#port-assignment)
    - [IP Assignemnt](#ip-assignemnt)
    - [Verification](#verification)
    - [Delete VLANS on file](#delete-vlans-on-file)
    - [Delete VLANS in memory](#delete-vlans-in-memory)
    - [Inter-VLAN Routing](#inter-vlan-routing)
+ [Trunks](#trunks)
    - [Create multi-switch vlan trunk](#create-multi-switch-vlan-trunk)
    - [Trunk Verification](#trunk-verification)
+ [EtherChannel](#etherchannel)
    - [Configure EtherChannel](#configure-etherchannel)
    - [Verify EtherChannel](#verify-etherchannel)
+ [DTP (Dynamic Trunking Protocol)](#dtp-dynamic-trunking-protocol)
    - [Configure DTP](#configure-dtp)
    - [Disable DTP](#disable-dtp)
    - [Verify DTP](#verify-dtp)
+ [OSPFv2](#ospfv2)
    - [Verify OSPF](#verify-ospf)
    - [Enable router OSPF process](#enable-router-ospf-process)
    - [Configure Loopback](#configure-loopback)
    - [Configure OSPF Router ID](#configure-ospf-router-id)
    - [Modify OSPF router ID](#modify-ospf-router-id)
    - [Configure OSPF With Network Command](#configure-ospf-with-network-command)
    - [Configure OSPF with `ip ospf`](#configure-ospf-with-ip-ospf)
    - [OSPF Passive Interfaces](#ospf-passive-interfaces)
    - [Find Designated Router and Backup](#find-designated-router-and-backup)
    - [Loopback and P2P Networks](#loopback-and-p2p-networks)
    - [Configure OSPF Priority](#configure-ospf-priority)
    - [Manually Set OSPF Link Cost](#manually-set-ospf-link-cost)
    - [Adjusting Reference Bandwidth](#adjusting-reference-bandwidth)
    - [Show OSPF Hello Packet Intervals](#show-ospf-hello-packet-intervals)
    - [Set OSPF Hello Packet Intervals](#set-ospf-hello-packet-intervals)
    - [Propogate Default Route](#ospf-propogate-default-route)
    - [Verify Propogated Default Route](#verify-propogated-default-route)
+ [Basic Security](#basic-security)
+ [Console Port](#console-port)
    - [Change Console Baudrate](#change-console-baudrate)
    - [Configure SSH](#configure-ssh)









### Setup
---
### Intialize

These commands wipe all config and reboot the device

```
erase startup-config
delete vlan.dat
reload
```

**Note:** Remeber to say "no" to saving running config on reload. If you say yes, running config will be saved and you wont be working with fresh config on reload.

#### Set Clock

*Show Clock*

```
show clock
```

*Sets clock to eastern US time*

```
clock timezone EST -5
```

*Revert to Default Timezone*

```
no clock timezone
```


### Interfaces
---



#### Assign Static IP to Interface

```
cont t
int g0/0
ip addr 10.0.0.10 255.255.255.0
```

#### Interface Ranges

`

*Select Single Range and Assign to a VLAN*
```
conf t
int range f0/1-12
switchport mode access
switch access vlan 10
end
```


### Interface Verification

```
show ip interface brief
```

#### Remove IP Addresses

```
conf t
int f0/1
no ip addr
end
```


### DHCP
---

#### Enable Router DHCP Server

This snippet configures a DHCP Server on R1 and will hand out
IPs on the `10.0.0.1/24` network.

```
conf t
ip domain name cisco.com
ip dhcp excluded-address 10.0.0.1
ip dhcp pool test
network 10.0.0.0 255.255.255.0
default-router 10.0.0.1
end
```


#### DHCP Verification

```
show running-config | section dhcp
shpw ip dhcp binding
show ip dhcp server statistics
```

#### Disable DHCP

```
conf t
no service dhcp
end
```

#### Re-enabled DHCP

```
conf t
service dhcp
end
```

#### Create VLAN DHCP

*Creates a Seperate DHCP Pool for each VLAN*

*Create VLANS*
```
conf t
vlan 10
name Management
vlan 20
name Sales
vlan 30
name Operations
end
```

*Configure SVI's and IP Address*

| VLAN | IP Address | Gateway
|------|------------|--------|
| 10   | 192.168.10.254 | 192.168.10.1
| 20 | 192.168.20.254 | 192.168.20.1|
| 30 | 192.168.30.254 | 192.168.30.1|

```
conf t
int vlan 10
ip address 192.168.10.254 255.255.255.0
ip default-gateway 192.168.10.1
no shut

int vlan 20
ip address 192.168.20.254 255.255.255.0
ip default-gateway 192.168.20.1
no shut

int vlan 30
ip address 192.168.30.254 255.255.255.0
ip default-gateway 192.168.30.1
no shut
end
```

*Add interfaces to VLANS, 8 ports per vlan*

```
conf t
int range f0/1-7
switchport mode access
switchport access vlan 10

int range f0/8-15
switchport mode access
switchport access vlan 20

int range f0/16-24
switchport mode access
switchport access vlan 30
end
```

*Create DHCP Pools for each vlan*

```
conf t
ip domain name cisco.com
ip dhcp excluded-address 192.168.10.1
ip dhcp pool vlan10pool
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
import all


ip dhcp excluded-address 192.168.20.1
ip dhcp pool vlan20pool
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
import all

ip dhcp excluded-address 192.168.30.1
ip dhcp pool vlan30pool
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
import all
end
```

#### Verify DHCP Pool

```
show ip dhcp pool
```

#### Delete DHCP Pool

```
conf t
no ip dhcp pool managementpool
end
```

### VLANs
---

#### VLAN Creation

```
conf t
vlan 10
name Faculty
exit
```

```
conf t
vlan 20
name Students
exit
```

#### Port Assignment

```
conf t
interface range Fa0/1-12
switchport mode access
switchport access vlan 10
end
```

```
conf t interface range Fa0/13-24
switchport mode access
switchport access vlan 20
end
```

```
conf t
interface Gi0/1
switchport mode access
switchport access vlan 99
end
```

#### IP Assignemnt

```
cont t
int vlan 99
ip address 10.0.0.1 255.255.255.0
end
```

#### Verification

```
show vlan brief
```


#### Delete VLANS on file

```
delete vlan.dat
```

#### Delete VLANS in memory
*Warning: Make sure you move ports to another vlan or the will be unsable*

```
conf t
no vlan 10
no vlan 20
end
```

#### Inter-VLAN Routing

```
conf t
interface G0/0/1.10
description Default Gateway for VLAN 10
encapsulation dot1Q 10
ip add 192.168.10.1 255.255.255.0
exit

interface G0/0/1.20
description Default Gateway for VLAN 20
encapsulation dot1Q 20
ip addr 192.168.20.1 255.255.255.0
exit

interface G0/0/1.99
description Default Gateway for VLAN 99
encapsulation dot1Q 99
ip addr 192.168.99.1 255.255.255.0
exit

interface G0/0/1
description Trunk link to S1
no shut
end
```

### Trunks
---

#### Create multi-switch vlan trunk

*S1*

```
conf t
interface Gi0/1
description Trunk Line to S2 Gi0/1
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 99
end
```

*Note: Remember to set the native vlan (to 99 for instance) on each switch in the trunk so you don't get a native vlan mismatch warning*

#### Trunk Verification

```
show interface trunk
show interface g0/1 switchport
```

### EtherChannel
---

Etherchannel protocols LACP and PAgP configure multiple physical interfaces and links to act as one logical one. You can configure up to 8 ports to act as a single link. This increases bandwidth and improves redundancy.


*Note: `mode active` sets the etherchannel group to use the LACP protocol*

#### Configure EtherChannel

*Configure etherchannel between two switches connected with two ethernet cables.*
```
conf t
int range f0/1-2
channel-group 1 mode active
exit
int port-channel 1
switchport mode trunk
switchport trunk allowed vlan 1,2,20
```


#### Verify EtherChannel

```
show interfaces trunk
show etherchannel summary
```

### DTP (Dynamic Trunking Protocol)
---


#### Configure DTP

```
conf t
int gi0/1
switchport mode dynamic auto
end
```

**or**

```
conf t
int gi0/1
switchport mode dynamic desirable
end
```


#### Disable DTP

*Usefull for connecting to devices that don't support Cisco propietary DTP or creating a static trunk*

```
conf t
int gi0/1
switchport mode trunk
switchport nonegotiate
end
```

#### Verify DTP

```
show dtp interface gi0/1
```

### OSPFv2
---
#### Verify OSPF
```
show ip ospf neighbour
show ip ospf database 
show ip protocols
```


##### Enable router OSPF process

Starting Mode: Global, Non-enabled

```
enable
conf t
router ospf 10
```

##### Configure Loopback

```
enable
conf t
interface Loopback 1
ip addr 1.1.1.1 255.255.255.255
end
```

##### Configure OSPF Router ID


```
conf t
router ospf 10
router-id 1.1.1.1
end
```

##### Modify OSPF router ID

_Prompt confirmation with 'y' needed_

```
conf t
router ospf 10
router-id 1.1.1.2
end
clear ip ospf process
```

_Verify_

```
show ip proto | include Router ID
```


##### Configure OSPF With Network Command

The following configures a trianngle of 3 routers connected to
each other as an OSPF point to point network.
`Router(config-router)# network network-address wildcard-mask area area-id`

```
conf t
router ospf 10
network 10.10.1.0 0.0.0.255 area 0
network 10.10.1.4 0.0.0.3 area 0
network 10.10.1.12 0.0.0.3 area 0
end
```

##### Configure OSPF with `ip ospf`

Configure OSPF directly on the interfaces rather with with the network
command.

Syntax: `Router(config-if)# ip ospf <process-id> area <area-id>`

```
R1(config-router)# interface GigabitEthernet 0/0/0
R1(config-if)# ip ospf 10 area 0
R1(config-if)# interface GigabitEthernet 0/0/1 
R1(config-if)# ip ospf 10 area 0
R1(config-if)# interface Loopback 0
R1(config-if)# ip ospf 10 area 0
R1(config-if)#
```

##### OSPF Passive Interfaces

```
conf t
router ospf 10
passive-interface loopback 0
end
```

```
conf t
router ospf 10
passive-interface Gi0/0/0
end
```

##### Find Designated Router and Backup

```
show ip ospf interface GigabitEthernet 0/0/0
```

##### Loopback and P2P Networks

Loobacks can be used to simulate real LAN networks

```
conf t
interface Loopback 0
ip ospf network point-to-point
```

```
show ip route | include 10.10.1
```

##### Configure OSPF Priority

```
conf t
int g0/0/1
ip ospf priority 255
end
```

Where `255` can be values from `0` to `255` with higher numbers making the router to be elected `DR`.

##### Manually Set OSPF Link Cost

```
conf t
int g0/0/1
ip ospf cost 25
interface l0
ip ospf cost 15
end
```
##### Adjusting Reference Bandwidth

```
Router# router ospf 10
Router(config-router) auto-cost reference bandwidth 1000
```

_Where 1000 is the speed of the link in Mpbs_
Common Values: 10, 100, 1000

##### Show OSPF Hello Packet Intervals

```
show ip ospf int g0/0/1
```

##### Set OSPF Hello Packet Intervals

```
Router(config-if)# ip ospf hello-interval <seconds>
```

```
conf t
int g0/0/1
ip ospf hello-interval 30
end
```

Note: dead-interval automatically gets set as `hello-interval * 4`

##### OSPF Propogate Default Route

```
conf t
ip route 0.0.0.0 0.0.0.0 loopback 1
router ospf 10
default-information originate
```

##### Verify Propogated Default Route

```
show ip route | begin Gateway
```

### Basic Security
---

```
conf t
enable secret class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
end
```


### Console Port
---
#### Change Console Baudrate

```
conf t
line con 0
speed 115200
end
```

```
conf t
line con 0
speed 9600
end
```


#### Configure SSH

```
show ip ssh
conf t
ip domain-name cisco.com
crypto key generate rsa

username admin secret ccna
line vty 0 15
transport input ssh
login local
exit
ip ssh version 2
exit
```

