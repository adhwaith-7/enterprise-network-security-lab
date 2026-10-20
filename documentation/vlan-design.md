# VLAN Design and Segmentation

## Overview

The network uses VLANs to logically separate different types of devices and users.

Each VLAN represents a different network or security zone.

This segmentation reduces unnecessary Layer 2 communication and provides a foundation for applying security policies between network zones.

---

## VLAN Structure

| VLAN ID | Name | Purpose | Network | Gateway |
|---:|---|---|---|---|
| 10 | EMPLOYEES | Employee/client systems | 192.168.10.0/24 | 192.168.10.1 |
| 20 | SERVERS | Internal services | 192.168.20.0/24 | 192.168.20.1 |
| 30 | GUEST | Guest systems | 192.168.30.0/24 | 192.168.30.1 |
| 40 | DMZ | Public-facing services | 192.168.40.0/24 | 192.168.40.1 |
| 99 | MANAGEMENT | Network administration | 192.168.99.0/24 | 192.168.99.1 |

---

## VLAN 10 - Employees

### Purpose

VLAN 10 is used for normal employee/client systems.

Network:

```text
192.168.10.0/24
```
Gateway:
```
192.168.10.1
```
Client devices receive their addresses through DHCP.
Example:
```
PC0
192.168.10.x

PC1
192.168.10.x
```
Employee systems can communicate with permitted internal services through R1.

### VLAN 20 - Servers

#### Purpose 

VLAN 20 contains internal infrastructure and application services.

Network:
```
192.168.20.0/24
```
Gateway:
```
192.168.20.1
```
Server0:
```
192.168.20.2
```
Services provided by Server0 include:
- DNS
- HTTP
The Server VLAN is protected from Guest access using an extended ACL.

### VLAN 30 - Guest

#### Purpose

VLAN 30 provides network access for guest devices.

Network:
```
192.168.30.0/24
```
Gateway:
```
192.168.30.1
```
Guest devices receive their addresses through DHCP.
Example:
```
PC3
192.168.30.x
```
#### Security Policy

Guest systems are not permitted to access the internal Server VLAN.
```
Guest → Server VLAN
DENY
```
Guest systems are still permitted to access the simulated Internet.
```
Guest → Internet
ALLOW
```
This demonstrates network segmentation combined with access-control enforcement.

### VLAN 40 - DMZ

#### Purpose

VLAN 40 is used as the DMZ network for services that may need to be accessed from outside the internal network.
Network:
```
192.168.40.0/24
```
Gateway:
```
192.168.40.1
```
DMZ Server:
```
192.168.40.2
```
The DMZ server provides HTTP service.
#### Security Policy

The DMZ is isolated from sensitive internal networks.
```
DMZ → Employee VLAN
DENY

DMZ → Server VLAN
DENY

DMZ → Internet
ALLOW
```
The web server is exposed externally through static NAT.

### VLAN 99 - Management 
#### Purpose 

VLAN 99 is dedicated to network-device management.

Network:
```
192.168.99.0/24
```
Gateway:
```
192.168.99.1
```
Switch management address:
```
192.168.99.2
```
Management PC:
```
192.168.99.10
```
SSH access to R1 and SW1 is restricted to this network.
This prevents normal user and guest networks from directly managing network infrastructure.

### Router-on-a-Stick 

R1 performs inter-VLAN routing using 802.1Q subinterfaces.

```
R1 G0/0.10 → VLAN 10
R1 G0/0.20 → VLAN 20
R1 G0/0.30 → VLAN 30
R1 G0/0.40 → VLAN 40
R1 G0/0.99 → VLAN 99
```
The physical connection between R1 and SW1 is configured as an 802.1Q trunk.
```
R1 G0/0
     |
     | 802.1Q Trunk
     |
SW1 G0/1
```
The trunk carries traffic for all required VLANs between the router and switch.

### Switch Port Assignment 

The switch uses access ports for endpoint devices.

| Switch Port | Connected Device | VLAN | Mode |
|---|---|---:|---|
| Fa0/1 | PC0 | 10 | Access |
| Fa0/2 | PC1 | 10 | Access |
| Fa0/3 | PC2 | 20 | Access |
| Fa0/4 | Server0 | 20 | Access |
| Fa0/5 | PC3 | 30 | Access |
| Fa0/6 | DMZ Server | 40 | Access |
| Fa0/7 | Management PC | 99 | Access |
| G0/1 | R1 | 10,20,30,40,99 | Trunk |

###  VLAN Isolation 

The VLAN architecture provides logical separation:
                 R1
                 |
             802.1Q Trunk
                 |
                SW1
                 |
    ┌────────────┼────────────┐
    │            │            │
 VLAN 10      VLAN 20      VLAN 30
Employees     Servers       Guest
                             
                 │
              VLAN 40
                DMZ

                 │
              VLAN 99
            Management
Devices within the same VLAN can communicate at Layer 2 according to normal switching behavior.
Communication between different VLANs must pass through R1.
This allows security policies to be enforced at the routing boundary.


