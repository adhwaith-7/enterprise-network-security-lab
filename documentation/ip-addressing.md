
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
