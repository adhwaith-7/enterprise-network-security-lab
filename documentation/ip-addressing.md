
# IP Addressing Plan

## Overview

This document defines the IPv4 addressing scheme used throughout the enterprise network.

Private IPv4 address space is used for internal VLANs, while dedicated addressing is used for the simulated WAN and Internet networks.

---

## VLAN Addressing

| VLAN | Purpose | Network | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| 10 | Employees | 192.168.10.0/24 | 255.255.255.0 | 192.168.10.1 |
| 20 | Servers | 192.168.20.0/24 | 255.255.255.0 | 192.168.20.1 |
| 30 | Guest | 192.168.30.0/24 | 255.255.255.0 | 192.168.30.1 |
| 40 | DMZ | 192.168.40.0/24 | 255.255.255.0 | 192.168.40.1 |
| 99 | Management | 192.168.99.0/24 | 255.255.255.0 | 192.168.99.1 |

---

## Infrastructure and Host Addresses

| Device | Role | IP Address | Network |
|---|---|---|---|
| R1 G0/0.10 | VLAN 10 Gateway | 192.168.10.1 | VLAN 10 |
| R1 G0/0.20 | VLAN 20 Gateway | 192.168.20.1 | VLAN 20 |
| R1 G0/0.30 | VLAN 30 Gateway | 192.168.30.1 | VLAN 30 |
| R1 G0/0.40 | VLAN 40 Gateway | 192.168.40.1 | VLAN 40 |
| R1 G0/0.99 | VLAN 99 Gateway | 192.168.99.1 | VLAN 99 |
| Server0 | Internal Server | 192.168.20.2 | VLAN 20 |
| DMZ Server | Web Server | 192.168.40.2 | VLAN 40 |
| SW1 | Management SVI | 192.168.99.2 | VLAN 99 |
| Management PC | Network Management | 192.168.99.10 | VLAN 99 |

---

## WAN Addressing

R1 and the simulated ISP are connected using a point-to-point /30 network.

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| R1 | G0/1 | 10.0.0.1 | 255.255.255.252 |
| ISP | G0/0 | 10.0.0.2 | 255.255.255.252 |

Network:

```text
10.0.0.0/30
```
Usable addresses:
```
10.0.0.1
10.0.0.2
```
### Simulated Internet Network

The ISP connects to a simulated external network.

| Device | Interface | IP Address | Subnet Mask | Gateway |
|---|---|---|---|---|
| ISP | G0/1 | 192.0.2.1 | 255.255.255.0 | — |
| Internet Server | NIC | 192.0.2.10 | 255.255.255.0 | 192.0.2.1 |

Network:
```
192.0.2.0/24
```
The 192.0.2.0/24 network is used as a documentation/test network to represent the simulated Internet in the Packet Tracer environment.

### DHCP Address Allocation

R1 acts as the DHCP server for the client VLANs.
The following address ranges are reserved:
```
192.168.10.1 - 192.168.10.20
192.168.20.1 - 192.168.20.20
192.168.30.1 - 192.168.30.20
```
Dynamic client addresses are therefore allocated from the remaining address space.

#### VLAN 10 
```
Network:       192.168.10.0/24
Gateway:       192.168.10.1
DHCP range:    192.168.10.21+
DNS:           192.168.20.2
```
#### VLAN 20
```
Network:       192.168.20.0/24
Gateway:       192.168.20.1
DHCP range:    192.168.20.21+
DNS:           192.168.20.2
```
#### VLAN 30
```
Network:       192.168.30.0/24
Gateway:       192.168.30.1
DHCP range:    192.168.30.21+
DNS:           192.168.20.2
```
### Static Addressing 

Infrastructure and service devices use static IP addresses.

Static addressing is used for:
- Router interfaces
- Switch management interface
- Internal server
- DMZ server
- Management PC
- ISP interfaces
- Internet server

This ensures that critical infrastructure and services remain reachable at predictable addresses.

### NAT Addressing 

Internal private networks use PAT when accessing the simulated Internet.

Translated internal networks:
```
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
```
The R1 WAN interface is used as the PAT address:
```
10.0.0.1
```
### DMZ Static NAT 

The DMZ web server uses:
```
Inside local:
192.168.40.2:80
```
It is published externally through:
```
Inside global:
10.0.0.1:80
```
This allows HTTP traffic from the simulated external network to reach the DMZ web server.

### Addressing Design Principles

The addressing plan follows these principles:
- Separate subnet for each security zone
- Dedicated management network
- Static addressing for infrastructure
- DHCP for normal client systems
- Reserved address space for gateways and infrastructure
- /30 subnet for the point-to-point WAN connection
- Separate external test network
- Predictable addressing for troubleshooting

### Summary

The addressing plan provides clear separation between users, servers, guests, DMZ systems, and management devices.

The structured addressing scheme also makes troubleshooting easier because the VLAN and security zone can be identified directly from an IP address.


