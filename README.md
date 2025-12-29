
# Enterprise VLAN Routing Lab with Multi-Router Static Routing

## Project Overview
This lab demonstrates enterprise-style network segmentation and connectivity using VLANs, router-on-a-stick, DHCP, and multi-router static routing in Cisco Packet Tracer. Two independent routed environments are first built, each containing multiple VLANs, and are then interconnected through a dedicated point-to-point /30 link to enable end-to-end communication between hosts across all routing domains.

This repository includes both the Cisco Packet Tracer project file used to build and validate the lab and a draw.io network topology diagram. The Packet Tracer file allows the full network configuration, device settings, and connectivity testing to be reviewed and reproduced. The draw.io diagram provides a high-level architectural view of the logical design, illustrating VLAN segmentation, switch-to-router trunking, router subinterfaces, static routing, and inter-router connectivity, helping visualize how all components interact end-to-end.

---

## Why These IP Ranges Were Chosen

### VLAN Subnets (/24)
Each VLAN uses a **/24 subnet (255.255.255.0)** because:
- A /24 is standard and simple for labs and enterprise access VLANs.
- It provides up to **254 usable host addresses** per VLAN (enough room for growth).
- It keeps addressing easy to recognize and troubleshoot.

Examples used in this lab:
- VLAN 10 → `192.168.10.0/24`
- VLAN 20 → `192.168.20.0/24`
- VLAN 10 (Network 2) → `192.168.100.0/24`
- VLAN 20 (Network 2) → `192.168.200.0/24`

The pattern “VLAN ID → matching subnet” (ex: VLAN 10 → 192.168.10.0/24) makes the design predictable and easier to document.

### Router-to-Router Link (/30)
The inter-router link uses **10.0.0.0/30 (255.255.255.252)** because:
- A /30 is ideal for point-to-point links: it provides **2 usable IPs** (one per router) with minimal waste.
- It is a common enterprise practice for WAN / router interconnect segments.

Usable addresses in `10.0.0.0/30`:
- Network: `10.0.0.0`
- Usable: `10.0.0.1`, `10.0.0.2`
- Broadcast: `10.0.0.3`

---

## Lab Objectives
- Create VLANs on Layer 2 switches
- Assign switch access ports to VLANs
- Configure a trunk from the switch to the router
- Configure router-on-a-stick using subinterfaces and 802.1Q tagging
- Provide DHCP scopes for each VLAN
- Build a second routed network with its own VLANs and DHCP
- Interconnect both routers using a /30 link
- Implement static routes to enable full inter-network communication
- Validate end-to-end reachability (PC-to-PC ping across both networks)

---

## Network Design Summary

### Network 1 (VLAN 10 / VLAN 20)
- VLAN 10 – `192.168.10.0/24`
- VLAN 20 – `192.168.20.0/24`

### Network 2 (VLAN 10 / VLAN 20)
- VLAN 10 – `192.168.100.0/24`
- VLAN 20 – `192.168.200.0/24`

### Router-to-Router Interconnect
- `10.0.0.0/30`
- Router A: `10.0.0.1`
- Router B: `10.0.0.2`

---

# Part 1 — Switch Configuration (Network 1)

## Step 1: Create VLANs
```bash
enable
configure terminal

vlan 10
 name MARKETING
exit

vlan 20
 name HR
exit
```

### What each command does
- `enable` — Enters privileged EXEC mode (required for configuration changes).
- `configure terminal` — Enters global configuration mode.
- `vlan 10` / `vlan 20` — Creates VLANs (or enters VLAN configuration if they already exist).
- `name MARKETING` / `name HR` — Assigns readable names for documentation and troubleshooting.
- `exit` — Leaves the current configuration sub-mode.

## Step 2: Assign Access Ports to VLANs
```bash
interface range fa0/1 - 2
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface range fa0/3 - 4
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
```

### What each command does
- `interface range fa0/1 - 2` — Selects multiple switch interfaces at the same time.
- `switchport mode access` — Forces the port to act as an access port (single VLAN only).
- `switchport access vlan 10` — Places the port into VLAN 10.
- `no shutdown` — Enables the port administratively.
- `exit` — Exits interface configuration mode.

## Step 3: Configure Trunk Port to the Router
```bash
interface gigabitEthernet0/1
 switchport mode trunk
 no shutdown
exit
end
write memory
```

### Why trunking is required
A trunk is required because the router must receive traffic for **multiple VLANs** over one physical cable. Trunking carries VLAN tags (802.1Q) so the router can separate VLAN 10 and VLAN 20 traffic.

### What each command does
- `interface gigabitEthernet0/1` — Selects the switch port connected to the router.
- `switchport mode trunk` — Enables trunking and allows multiple VLANs over the same link.
- `no shutdown` — Ensures the trunk port is enabled.
- `end` — Exits configuration mode back to privileged EXEC.
- `write memory` — Saves the running configuration to startup configuration.

<img width="1913" height="987" alt="image" src="https://github.com/user-attachments/assets/04644d83-0363-4985-84fe-d428c497f917" />

---

# Part 2 — Router Configuration (Network 1)

## Step 1: Prepare the Physical Interface (Router-on-a-Stick)
```bash
enable
configure terminal

interface gigabitEthernet0/0/0
 no ip address
 no shutdown
exit
```

### Why “no ip address” is used
In router-on-a-stick, **the physical interface is only the carrier** for VLAN-tagged traffic. IP addressing is placed on **subinterfaces**, not the physical interface.

### What each command does
- `interface gigabitEthernet0/0/0` — Selects the router interface connected to the switch trunk.
- `no ip address` — Ensures the physical interface has no Layer 3 address (subinterfaces will handle L3).
- `no shutdown` — Enables the interface.

## Step 2: Create Subinterfaces for VLAN 10 and VLAN 20
```bash
interface gigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
```

### Why subinterfaces are used
A single physical link can serve multiple VLANs by creating **logical interfaces** (subinterfaces). Each subinterface acts as the **default gateway** for its VLAN.

### What each command does
- `interface gigabitEthernet0/0/0.10` — Creates/enters subinterface for VLAN 10.
- `encapsulation dot1Q 10` — Associates the subinterface with VLAN 10 using 802.1Q tagging.
- `ip address 192.168.10.1 255.255.255.0` — Assigns the VLAN 10 default gateway.
- Same logic applies for VLAN 20 (`.20`, VLAN tag 20, gateway 192.168.20.1).

## Step 3: DHCP Exclusions
```bash
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
```

### Why exclusions are required
Exclusions prevent DHCP from issuing addresses reserved for:
- Default gateways
- Infrastructure devices
- Static assignments

### What each command does
- `ip dhcp excluded-address <start> <end>` — Marks a range as “do not lease.”

## Step 4: DHCP Pools
```bash
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
exit

ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
exit
end
write memory
```

### Why DHCP pools are required per VLAN
Each VLAN is a separate Layer 3 network. DHCP must provide:
- An IP address in the correct subnet
- The correct default gateway for that VLAN

### What each command does
- `ip dhcp pool VLAN10` — Creates a DHCP scope named VLAN10.
- `network 192.168.10.0 255.255.255.0` — Defines which subnet addresses will be leased from.
- `default-router 192.168.10.1` — Sets the gateway delivered to clients.
- `dns-server 8.8.8.8` — Sets DNS server delivered to clients.
- `end` / `write memory` — Exits and saves.

<img width="1917" height="977" alt="image" src="https://github.com/user-attachments/assets/3b57ad04-99f8-4bc3-9791-5527d4d7ecf3" />
<img width="387" height="331" alt="image" src="https://github.com/user-attachments/assets/87ac6168-1c4c-4fb1-990e-dc79aa9c7faf" />

---

# Part 3 — Switch & Router Configuration (Network 2) ✅ (Added Back)

## Switch Commands (Network 2)
```bash
enable
configure terminal

vlan 10
 name VLAN10
exit

vlan 20
 name VLAN20
exit

interface range fa0/1 - 2
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface range fa0/3 - 4
 switchport mode access
 switchport access vlan 20
 no shutdown
exit

interface gig0/1
 switchport mode trunk
 no shutdown
exit
end
write memory
```

### What each command does (Network 2 switch)
- `vlan 10` / `vlan 20` — Creates VLANs used to segment endpoints.
- `interface range fa0/1 - 2` — Assigns two access ports to VLAN 10.
- `interface range fa0/3 - 4` — Assigns two access ports to VLAN 20.
- `interface gig0/1` + `switchport mode trunk` — Builds the trunk link to carry both VLANs to the router.

<img width="1918" height="966" alt="image" src="https://github.com/user-attachments/assets/5754d13e-e948-4c72-a08d-04ae908128ae" />

## Router Commands (Network 2)
```bash
enable
configure terminal

interface gig0/0/0
 no ip address
 no shutdown
exit

interface gig0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.100.1 255.255.255.0
exit

interface gig0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.200.1 255.255.255.0
exit

ip dhcp excluded-address 192.168.100.1 192.168.100.10
ip dhcp excluded-address 192.168.200.1 192.168.200.10

ip dhcp pool VLAN10
 network 192.168.100.0 255.255.255.0
 default-router 192.168.100.1
 dns-server 8.8.8.8
exit

ip dhcp pool VLAN20
 network 192.168.200.0 255.255.255.0
 default-router 192.168.200.1
 dns-server 8.8.8.8
exit
end
write memory
```

### What changes in Network 2
Only the IP ranges change to keep Network 2 unique:
- VLAN 10 gateway becomes `192.168.100.1`
- VLAN 20 gateway becomes `192.168.200.1`

<img width="1918" height="971" alt="image" src="https://github.com/user-attachments/assets/8c3d8e60-bfe9-42e7-969f-2e9be2fd6e7d" />
<img width="1915" height="972" alt="image" src="https://github.com/user-attachments/assets/1f872a88-aa0f-43e7-85d7-0a8783accd06" />
<img width="332" height="257" alt="image" src="https://github.com/user-attachments/assets/a362c7a1-02f5-4618-adc0-a4979bfdd28a" />

---

# Part 4 — DHCP Validation on PCs ✅ (Preserved)

### Network 1 PCs (DHCP Enabled)
<img width="1915" height="1017" alt="image" src="https://github.com/user-attachments/assets/3ca3803c-d75d-4b1b-9505-ef629b852a46" />
<img width="1908" height="1011" alt="image" src="https://github.com/user-attachments/assets/0d1d2a69-2b66-4d42-a845-c4419a3c6035" />

### Network 2 PCs (DHCP Enabled)
<img width="1917" height="1002" alt="image" src="https://github.com/user-attachments/assets/792b927e-f024-464b-8452-43e1fd010d8a" />
<img width="1918" height="978" alt="image" src="https://github.com/user-attachments/assets/c7f4fb32-7224-494e-9e1e-29b938cd87fe" />

---

# Part 5 — Interconnect Both Routers (Static Routing)

<img width="785" height="388" alt="image" src="https://github.com/user-attachments/assets/eadb0f6d-5dbc-415e-9f76-fb251af3da2c" />

## Why static routing is required
Each router only knows:
- Its directly-connected VLAN subnets
- Its directly-connected router-link subnet

Static routes are required so each router knows how to reach the **remote VLAN networks** behind the other router. Static routes must exist on **both routers** to enable **bidirectional** communication.

## Router A (First Router) — Interconnect Configuration
```bash
interface gigabitEthernet0/0/0
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit

interface gigabitEthernet0/0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

ip route 192.168.100.0 255.255.255.0 10.0.0.2
ip route 192.168.200.0 255.255.255.0 10.0.0.2
end
write memory
```

### What each command does
- `ip address 192.168.30.1 255.255.255.0` — Assigns L3 addressing for the local interconnect LAN stage as documented.
- `ip address 10.0.0.1 255.255.255.252` — Assigns the /30 point-to-point address on the router-to-router link.
- `ip route 192.168.100.0 ... 10.0.0.2` — Forwards traffic for VLAN 10 (Network 2) toward Router B.
- `ip route 192.168.200.0 ... 10.0.0.2` — Forwards traffic for VLAN 20 (Network 2) toward Router B.

<img width="1908" height="981" alt="image" src="https://github.com/user-attachments/assets/90e7470c-b65b-4164-8a65-89e67b0abbb3" />

## Router B (Second Router) — Interconnect Configuration
```bash
interface gigabitEthernet0/0/0
 ip address 192.168.40.1 255.255.255.0
 no shutdown
exit

interface gigabitEthernet0/0/1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
end
write memory
```

### What each command does
- `ip address 192.168.40.1 255.255.255.0` — Assigns L3 addressing for the local interconnect LAN stage as documented.
- `ip address 10.0.0.2 255.255.255.252` — Assigns the /30 point-to-point address on the router-to-router link.
- `ip route 192.168.10.0 ... 10.0.0.1` — Forwards traffic for VLAN 10 (Network 1) toward Router A.
- `ip route 192.168.20.0 ... 10.0.0.1` — Forwards traffic for VLAN 20 (Network 1) toward Router A.

<img width="1917" height="981" alt="image" src="https://github.com/user-attachments/assets/a7999a33-f2f4-4675-a316-37af9b9db9b6" />

---

# Final Topology and Validation

<img width="1918" height="1022" alt="image" src="https://github.com/user-attachments/assets/cae2ac0d-f13c-4907-9c4e-255d452e60b2" />

## End-to-End Connectivity Testing (PC-to-PC)
<img width="1906" height="971" alt="image" src="https://github.com/user-attachments/assets/87d457d6-e828-46d2-b92f-af74e2fc513f" />
<img width="1918" height="1017" alt="image" src="https://github.com/user-attachments/assets/e8b66e5a-284f-41d5-83a5-f3d55129230b" />

---

## Operational Notes (Troubleshooting Logic)

### DHCP Failed / APIPA (169.254.x.x)
- APIPA indicates the DHCP request did not reach the DHCP server.
- Common causes:
  - PC port not placed in the correct VLAN
  - Switch-to-router link not configured as a trunk
  - Router subinterface encapsulation mismatch
  - Router interface administratively down

### One-Way Ping (Asymmetric Routing)
- If one network can ping the other but not vice versa, the most common cause is missing routes on one router.
- Static routing must exist on **both routers** for bidirectional communication.

---

## Conclusion
This project demonstrates enterprise network segmentation and routing using VLANs and router-on-a-stick, automated host addressing through DHCP, and inter-network connectivity using static routing across a /30 point-to-point link. The final validation confirms full bidirectional communication between hosts across both routed networks and all VLAN segments.

---

## Appendix — Command Reference and Behavior (Quick Study Guide)

### Privilege and Configuration Modes
- `enable` — Enters privileged EXEC mode.
- `configure terminal` — Enters global configuration mode.
- `exit` — Moves up one configuration level.
- `end` — Exits configuration mode and returns to privileged EXEC.
- `write memory` — Saves the running configuration to startup configuration.

### Verification Commands
- `show ip interface brief` — Displays interface status and IP assignments.
- `show ip route` — Displays the routing table.
- `show vlan brief` — Confirms VLAN creation and port membership.
- `show interfaces trunk` — Verifies trunk operation.
- `ping <destination>` — Tests Layer 3 connectivity and routing logic.
