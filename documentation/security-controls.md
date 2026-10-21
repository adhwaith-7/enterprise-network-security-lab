# Network Security Controls

## Overview

Security controls were implemented throughout the network to reduce unnecessary access, isolate different network zones, protect network infrastructure, and control traffic between trusted and untrusted segments.

The main security controls implemented in this project are:

- VLAN segmentation
- Extended ACLs
- Guest network isolation
- DMZ isolation
- NAT/PAT
- Static NAT for the DMZ web server
- SSH-only device management
- Management VLAN
- SSH access restrictions
- Switch port security
- Unused-port shutdown
- Secure device credentials

---

## VLAN Segmentation

The network is divided into separate VLANs based on function.

| VLAN | Security Zone |
|---:|---|
| 10 | Employees |
| 20 | Internal Servers |
| 30 | Guest |
| 40 | DMZ |
| 99 | Management |

This prevents the network from operating as one flat Layer 2 broadcast domain.

Inter-VLAN traffic must pass through R1, where routing and access-control policies can be applied.

---

## Guest Network Isolation

Guest devices are placed in VLAN 30.

Network:

```text
192.168.30.0/24
```
An extended ACL prevents Guest systems from accessing the internal Server VLAN.
```
ip access-list extended GUEST-RESTRICTIONS
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip any any
```
The ACL is applied inbound on the Guest subinterface:
```
interface gigabitEthernet 0/0.30
 ip access-group GUEST-RESTRICTIONS in
```
Result
```
Guest → Internal Servers    BLOCKED
Guest → Internet            ALLOWED
```
This provides Internet access without allowing guests to directly access internal server resources.

## DMZ Isolation

The DMZ is implemented using VLAN 40.

Network:
```
192.168.40.0/24
```
DMZ Server:
```
192.168.40.2
```
The DMZ ACL prevents the DMZ server network from initiating traffic toward sensitive internal networks.
```
ip access-list extended DMZ-RESTRICTIONS
 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip any any
```
The ACL is applied inbound on the DMZ subinterface:
```
interface gigabitEthernet 0/0.40
 ip access-group DMZ-RESTRICTIONS in
```
Result
```
DMZ → Employees     BLOCKED
DMZ → Servers       BLOCKED
DMZ → Internet      ALLOWED
```
This demonstrates the principle of isolating externally exposed services from internal systems.

## NAT/PAT 

Private internal addresses are translated before accessing the simulated Internet.

The internal networks included in NAT are:
```
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
```
PAT uses the R1 WAN interface:
```
ip nat inside source list 1 interface gigabitEthernet 0/1 overload
```
The internal VLAN interfaces are marked as NAT inside interfaces, while R1 G0/1 is marked as the NAT outside interface.
#### Purpose
PAT allows multiple internal hosts to share the R1 WAN address when accessing the simulated Internet.

## Static NAT for DMZ Web Server 

The DMZ web server is published externally using static NAT.
```
ip nat inside source static tcp 192.168.40.2 80 10.0.0.1 80
```
Traffic flow:
```
External Client
      |
      | TCP/80
      ↓
10.0.0.1:80
      |
      ↓
192.168.40.2:80
      |
      ↓
DMZ Web Server
```
This allows HTTP access to the DMZ service without directly exposing the internal Server VLAN.

## Dedicated Management VLAN 

Network-device management is separated into VLAN 99.

Network:
```
192.168.99.0/24
```
R1:
```
192.168.99.1
```
SW1:
```
192.168.99.2
```
Management PC:
```
192.168.99.10
```
This separates administrative traffic from normal employee, guest, server, and DMZ traffic.

## SSH Device Management

R1 and SW1 are configured for SSH-based remote management.

SSH version 2 is enabled:
```
ip ssh version 2
```
Local administrative authentication is configured using:
```
username admin privilege 15 secret <configured-secret>
```
VTY lines use local authentication:
```
login local
```
Only SSH is permitted:
```
transport input ssh
```
Telnet is therefore not accepted on the configured VTY lines.

## SSH Access Restriction

SSH access is restricted to the Management VLAN using a standard access list.
```
ip access-list standard MGMT-SSH
 permit 192.168.99.0 0.0.0.255
 deny any
```
The ACL is applied to the VTY lines:
```
access-class MGMT-SSH in
```
Result
```
Management VLAN → R1 SSH     ALLOWED
Management VLAN → SW1 SSH    ALLOWED
Other networks → SSH          RESTRICTED
```
This reduces the exposure of network-device management services.

## Switch Port Security

Port security is enabled on access ports.

Configuration characteristics:
```
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation restrict
```
Configuration
- Maximum MAC addresses: 1
- Sticky MAC learning: Enabled
- Violation mode: Restrict

Sticky MAC learning allows the switch to dynamically learn the expected device MAC address.
If an unauthorized MAC address appears, restrict mode drops the violating frames while keeping the interface operational.

## Unused Port Shutdown

Unused switch ports are administratively disabled.

Configured unused ports include:
```
FastEthernet 0/8 - 24
GigabitEthernet 0/2
```
The ports were assigned a descriptive label:
```
description UNUSED_DISABLED
```
This reduces the possibility of unauthorized devices being connected to unused physical interfaces.

## Secure Device Credentials 

Administrative access uses locally configured credentials and privileged EXEC secrets.

R1 and SW1 use:
```
enable secret
```
and local user authentication:
```
username admin privilege 15 secret
```
Password encryption is also enabled on SW1:
```
service password-encryption
```
## Default Route and Controlled Internet Access 

R1 uses a default route toward the simulated ISP:
```
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```
This provides a controlled path from internal networks toward the simulated Internet.
The combination of routing, NAT, and ACL policies determines which networks can access external resources.

## Security Policy Summary 

| Source | Destination | Result |
|---|---|---|
| Employees | Servers | Allowed |
| Employees | Internet | Allowed |
| Guest | Servers | Blocked |
| Guest | Internet | Allowed |
| DMZ | Employees | Blocked |
| DMZ | Servers | Blocked |
| DMZ | Internet | Allowed |
| Management | R1 SSH | Allowed |
| Management | SW1 SSH | Allowed |
| Other networks | R1/SW1 SSH | Restricted |

## Security Validation 

The implemented controls were tested through deliberate connectivity tests.

#### Guest Isolation 
```
Guest PC → Server0
```
Expected:
```
BLOCKED
```
#### Guest Internet Access 
```
Guest PC → Internet Server
```
Expected:
```
ALLOWED
```
#### DMZ Isolation 
```
DMZ Server → Server0
```
Expected:
```
BLOCKED
```
#### DMZ Internet Access 
```
DMZ Server → Internet Server
```
Expected:
```
ALLOWED
```
#### Management Access 
```
Management PC → SW1 SSH
Management PC → R1 SSH
```
Expected:
```
ALLOWED
```
#### Security Design Principles
The implementation demonstrates several practical security principles:
#### Segmentation
Separate network zones limit unnecessary communication.
#### Least Privilege 
Traffic between security zones is permitted only where required.
#### Management Isolation 
Network-device administration is separated from normal user traffic.
#### Defense in Depth 
Multiple controls are used together rather than relying on a single security mechanism.
#### Reduced Attack Surface 
Unused switch ports are disabled and device management is restricted to SSH and the Management VLAN.
#### Controlled Exposure 
The DMZ allows externally accessible services while protecting internal networks from direct DMZ access.

#### Summary 

The security architecture combines Layer 2 segmentation, Layer 3 access control, NAT, DMZ isolation, secure management, and switch-level protection.
The controls were not only configured but also validated through connectivity testing and deliberate failure scenarios.



















