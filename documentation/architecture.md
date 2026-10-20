# Network Architecture

## Overview

This project implements a segmented enterprise network using Cisco Packet Tracer.

The architecture separates users, servers, guests, externally exposed services, and network management into different VLANs.

Inter-VLAN communication is handled by Router R1 using Router-on-a-Stick.

R1 also provides DHCP services, NAT/PAT, access-control policies, and secure remote management.

---

## Physical Architecture

The main network devices are:

- R1 - Cisco 2911 Router
- SW1 - Cisco 2960 Switch
- ISP - Cisco 2911 Router
- Employee PCs
- Server0
- Guest PC
- DMZ Server
- Management PC
- Internet Server

The primary LAN connection is:

R1 → 802.1Q Trunk → SW1

The WAN connection is:

R1 → ISP → Internet Server

---

## Logical Architecture

```text
                         INTERNET
                            |
                     Internet Server
                       192.0.2.10
                            |
                        ISP Router
                            |
                       10.0.0.0/30
                            |
                       R1 Router
                            |
                       802.1Q Trunk
                            |
                         SW1
             ______________|_______________
            |       |       |       |      |
         VLAN 10 VLAN 20 VLAN 30 VLAN 40 VLAN 99
         Employee Servers  Guest    DMZ   Management
            |       |       |       |      |
           PCs    Server0   PC3   DMZ     Mgmt PC
                                  Server
```
## Router-on-a-Stick

R1 performs inter-VLAN routing using subinterfaces on GigabitEthernet0/0.

Each VLAN is assigned its own subinterface and gateway.
```
| Interface | VLAN |   Gateway    |
|-----------|------|--------------|
|  G0/0.10  |  10  | 192.168.10.1 |
|  G0/0.20  |  20  | 192.168.20.1 |
|  G0/0.30  |  30  | 192.168.30.1 |
|  G0/0.40  |  40  | 192.168.40.1 |
|  G0/0.99  |  99  | 192.168.99.1 |
```
The physical G0/0 interface operates as the trunk connection to SW1.

## Switch Architecture
