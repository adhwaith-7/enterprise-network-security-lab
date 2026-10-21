# Network Troubleshooting and Failure Analysis

## Overview

This project includes deliberate network failures to demonstrate a structured troubleshooting methodology.

Instead of immediately changing configurations, each failure was investigated using a repeatable process:

```text
SYMPTOM
   ↓
HYPOTHESIS
   ↓
TEST
   ↓
EVIDENCE
   ↓
ROOT CAUSE
   ↓
FIX
   ↓
VERIFY
   ↓
DOCUMENT
```
Seven different failure scenarios were introduced, investigated, corrected, and verified.

### 01 - DNS Failure 

#### Objective 
Demonstrate how to distinguish a DNS resolution problem from a general network connectivity problem.
#### Failure Introduced 
The client DNS configuration was changed from the correct internal DNS server to an incorrect address.

Incorrect DNS server:
```
192.168.20.99
```
Correct DNS server:
```
192.168.20.2
```
#### Symptom 

The hostname:
```
server.company.local
```
could not be resolved.
However, direct IP connectivity to Server0 remained functional.

#### Tests 

Hostname test:
```
ping server.company.local
```
Direct IP test:
```
ping 192.168.20.2
```
Evidence
```
Hostname resolution → FAILED
Direct IP connectivity → WORKING
```
This indicated that the underlying network path was operational.

#### Root Cause 
Incorrect DNS server configuration on the client.

#### Fix 

The DNS server was restored to:
```
192.168.20.2
```

#### Verification 
Hostname resolution and connectivity were tested again.

#### Lesson 
When hostname-based communication fails but direct IP communication works, DNS should be investigated before assuming a routing or connectivity failure.

### 02 - DHCP Failure 

#### Objective 
Demonstrate troubleshooting of client IP address assignment.

### Failure Introduced
The VLAN 10 DHCP pool was removed from R1.

Command used:
```
no ip dhcp pool VLAN10-DHCP
```

#### Symptom 
A VLAN 10 client was unable to obtain an IP address after forcing a new DHCP request.

#### Investigation 
The DHCP configuration on R1 was inspected.
The expected pool was missing.

#### Root Cause
The VLAN 10 DHCP pool had been removed.

#### Fix 

The DHCP pool was recreated:
```
ip dhcp excluded-address 192.168.10.1 192.168.10.20

ip dhcp pool VLAN10-DHCP
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.20.2
```
#### Verification
The client was switched back to DHCP and successfully received an address from the VLAN 10 pool.

#### Lesson
When a client cannot obtain an IP address, verify the DHCP pool, network, gateway, exclusions, and client configuration. 

### 03 - VLAN / Trunk Failure 

#### Objective 
Demonstrate troubleshooting of a failed router-to-switch trunk.

#### Failure Introduced
SW1 G0/1 was changed from trunk mode to access mode.
```
interface gigabitEthernet 0/1
 switchport mode access
```
#### Symptom
The clients could no longer communicate through the router.

Example:
```
PC0 → 192.168.10.1
100% packet loss
```
Inter-VLAN connectivity also failed.

#### Investigation
The trunk configuration was checked:
```
show interfaces trunk
```
No active trunk was displayed.

#### Root Cause
The R1-to-SW1 link was no longer operating as an 802.1Q trunk. 

#### Fix
The interface was restored: 
```
interface gigabitEthernet 0/1
 switchport mode trunk
```
#### Verification
The trunk was checked again: 
```
show interfaces trunk
```
Inter-VLAN connectivity was then restored.

#### Lesson
When Router-on-a-Stick stops working across multiple VLANs, the physical interface mode and trunk status should be checked early. 

### 04 - ACL Failure 

#### Objective
Demonstrate troubleshooting of ACL rule ordering and first-match processing. 

#### Baseline
Before introducing the failure: 
```
Guest → Server VLAN
BLOCKED
Guest → Internet
ALLOWED
```
#### Failure Introduced
An additional rule was introduced to block Guest access to the simulated Internet.
The ACL was temporarily configured as: 
```
deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
deny ip 192.168.30.0 0.0.0.255 192.0.2.0 0.0.0.255
permit ip any any
```
#### Symptom
Guest access to the Internet stopped working. 

#### Investigation
The ACL was inspected using: 
```
show access-lists
```
The deny statements showed matching traffic.

#### Root Cause
ACL entries are processed from top to bottom using first-match logic.
The newly introduced rule intentionally changed the Guest traffic policy.
The exercise demonstrated how rule placement and ordering directly affect traffic behavior. 

#### Fix
The unintended Internet deny rule was removed.
Final intended policy: 
```
deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
permit ip any any
```
#### Verification 
```
Guest → Server VLAN
BLOCKED

Guest → Internet
ALLOWED
```
#### Lesson 
ACL troubleshooting requires checking both the configured rules and their order.
A syntactically correct ACL can still produce an incorrect security policy.

### 05 - NAT Failure

#### Objective
Demonstrate troubleshooting of Internet connectivity caused by missing PAT configuration.

#### Baseline
Before the failure:
```
PC0 → Internet Server
SUCCESSFUL
```
#### Failure Introduced
The PAT rule was removed: 
```
no ip nat inside source list 1 interface gigabitEthernet 0/1 overload
```
#### Symptom
Internal clients could no longer reach the simulated Internet.
Example: 
```
ping 192.0.2.10
100% packet loss
```
#### Investigation
NAT configuration was checked: 
```
show running-config | include ip nat
```
NAT translations were also checked:
```
show ip nat translations
```
The expected PAT rule was missing.

#### Root Cause
The dynamic PAT configuration had been removed. 

#### Fix
PAT was restored: 
```
ip nat inside source list 1 interface GigabitEthernet0/1 overload
```
#### Verification
NAT translations appeared again: 
```
show ip nat translations
```
Internet connectivity was restored.

#### Lesson
When private internal hosts cannot reach an external network, verify routing first and then confirm NAT/PAT configuration and translations. 

### 06 - Port Security Failure 

#### Objective
Demonstrate Layer 2 security enforcement using switch port security. 

#### Baseline
Fa0/1 was configured with: 
```
Maximum MAC addresses: 1
Sticky MAC: Enabled
Violation mode: Restrict
```
#### Failure Introduced
A different endpoint MAC address was temporarily connected to the protected port. 

#### Symptom
The switch detected MAC address violations.

#### Investigation
The interface was checked using: 
```
show port-security interface fa0/1
```
The output showed:
```
Port Security: Enabled
Port Status: Secure-up
Violation Mode: Restrict
Maximum MAC Addresses: 1
Sticky MAC Addresses: 1
Security Violation Count: 7
```
The last source MAC address differed from the previously learned sticky MAC.

#### Root Cause
An unauthorized/different MAC address appeared on a port configured to allow only one learned MAC address. 

#### Fix
The original endpoint was restored to Fa0/1.

#### Verification
The original topology was restored and the protected port continued operating under the configured port-security policy. 

#### Lesson
Port security can detect unexpected devices at the access layer and restrict their traffic without necessarily shutting down the interface. 

### 07 - Default Route / Routing Failure 

#### Objective
Demonstrate troubleshooting of loss of external connectivity caused by a missing default route. 

#### Baseline
Before the failure: 
```
PC0 → Internet Server
SUCCESSFUL
```
#### Failure Introduced
The default route on R1 was removed: 
```
no ip route 0.0.0.0 0.0.0.0 10.0.0.2
```
#### Symptom
Internal hosts could no longer reach the simulated Internet. 

#### Investigation
The routing table was inspected: 
```
show ip route
```
The router reported:
```
Gateway of last resort is not set
```
The default route was absent.

#### Root Cause
R1 had no route for destinations outside its directly connected networks. 

#### Fix
The default route was restored: 
```
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```
#### Verification
The routing table showed the default route again.
Internet connectivity was tested successfully. 

#### Lesson
When external connectivity fails while internal networks remain reachable, checking for a valid default route is an important troubleshooting step.

### Troubleshooting Summary 

| Scenario | Failure | Root Cause | Verification |
|---|---|---|---|
| DNS | Hostname resolution failed | Incorrect DNS server | Ping hostname and IP |
| DHCP | Client received no address | Missing DHCP pool | DHCP lease |
| Trunk | VLAN connectivity failed | Trunk changed to access | `show interfaces trunk` |
| ACL | Guest Internet blocked | ACL policy/order | `show access-lists` |
| NAT | Internet unreachable | PAT rule removed | `show ip nat translations` |
| Port Security | MAC violations | Different MAC detected | `show port-security` |
| Routing | Internet unreachable | Default route removed | `show ip route` |

### Troubleshooting Methodology 

The project follows a repeatable troubleshooting process: 

1. Identify the Symptom
   Determine exactly what is failing.
2. Establish a Baseline
   Compare the failure against previously verified behavior.
3. Form a Hypothesis
   Identify the most likely layer or component responsible.
4. Test the Hypothesis
   Use targeted commands and connectivity tests.
5. Collect Evidence
   Use command output and observed behavior rather than assumptions.
6. Identify Root Cause
   Determine the configuration or component responsible for the failure.
7. Apply the Fix
   Make the smallest appropriate configuration change.
8. Verify
   Repeat the original test and confirm normal operation.
9. Document
   Record the symptom, evidence, root cause, fix, and result.

### Key Troubleshooting Commands 

The following commands were useful throughout the project:
```
show ip interface brief
show ip route
show vlan brief
show interfaces trunk
show access-lists
show ip nat translations
show running-config | include ip nat
show port-security interface fa0/1
show ip dhcp binding
show ip dhcp pool
```
These commands provide visibility into Layer 2, Layer 3, DHCP, ACL, NAT, and switch-security states.

### Conclusion 

The troubleshooting portion of the project demonstrates that network engineering is not limited to initial configuration.

A functional network must also be monitored, tested, diagnosed, repaired, and validated when failures occur.

The seven deliberate failure scenarios provided practical experience with:
- DNS troubleshooting
- DHCP troubleshooting
- VLAN and trunk troubleshooting
- ACL troubleshooting
- NAT troubleshooting
- Layer 2 security troubleshooting
- Routing troubleshooting

The process emphasized evidence-based diagnosis rather than trial-and-error configuration.



















