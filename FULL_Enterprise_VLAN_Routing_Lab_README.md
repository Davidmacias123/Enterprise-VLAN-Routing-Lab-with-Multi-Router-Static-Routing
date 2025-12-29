
# Enterprise VLAN Routing Lab with Multi-Router Static Routing

## Project Overview
This project documents the design and implementation of an enterprise-style network using VLAN segmentation, router-on-a-stick, DHCP, and multi-router static routing in Cisco Packet Tracer. Two independent routed networks are first built and validated, then interconnected using a dedicated point-to-point link to enable full end-to-end communication between hosts across all VLANs and routing domains.

---

## Why These IP Addresses Were Chosen

### VLAN Networks (/24)
Each VLAN uses a /24 subnet (255.255.255.0) because:
- It is the most common enterprise access-layer subnet size
- It supports up to 254 usable host addresses per VLAN
- It simplifies troubleshooting and documentation
- It scales well for small-to-medium enterprise environments

The lab intentionally aligns VLAN IDs with IP addressing to make the design intuitive:
- VLAN 10 → 192.168.10.0/24
- VLAN 20 → 192.168.20.0/24
- VLAN 10 (Network 2) → 192.168.100.0/24
- VLAN 20 (Network 2) → 192.168.200.0/24

### Router-to-Router Link (/30)
The inter-router link uses a /30 subnet (255.255.255.252) because:
- It is optimized for point-to-point links
- It provides exactly two usable IP addresses
- It minimizes IP address waste
- It reflects real-world WAN and router interconnect design

---

## Lab Objectives
- Design VLAN-based segmentation using Layer 2 switches
- Assign access ports to specific VLANs
- Configure trunking between switches and routers
- Implement router-on-a-stick with 802.1Q subinterfaces
- Provide DHCP services for each VLAN
- Build two independent routed networks
- Interconnect routers using a point-to-point /30 link
- Configure static routes for bidirectional communication
- Validate end-to-end connectivity between all hosts

---

## Switch Configuration – Network 1

### VLAN Creation
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

**Command Explanation**
- `enable`: Enters privileged EXEC mode.
- `configure terminal`: Enters global configuration mode.
- `vlan <id>`: Creates or enters a VLAN.
- `name <name>`: Assigns a descriptive VLAN name.
- `exit`: Exits the current configuration context.

### Access Port Assignment
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

**Command Explanation**
- `interface range`: Selects multiple interfaces simultaneously.
- `switchport mode access`: Forces the port into access mode.
- `switchport access vlan <id>`: Assigns the port to a VLAN.
- `no shutdown`: Enables the interface administratively.

### Trunk Port to Router
```bash
interface gigabitEthernet0/1
 switchport mode trunk
 no shutdown
exit
end
write memory
```

**Why Trunking Is Required**
A trunk allows multiple VLANs to traverse a single physical link. This is required for router-on-a-stick so the router can receive tagged traffic from multiple VLANs.

<img src="https://github.com/user-attachments/assets/04644d83-0363-4985-84fe-d428c497f917" />

---

## Router Configuration – Network 1

### Physical Interface Preparation
```bash
interface gigabitEthernet0/0/0
 no ip address
 no shutdown
exit
```

**Explanation**
The physical interface carries tagged VLAN traffic only. Layer 3 addressing is handled by subinterfaces.

### Subinterface Configuration
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

**Explanation**
Each subinterface represents a VLAN gateway. 802.1Q encapsulation ensures traffic is correctly tagged and separated.

### DHCP Configuration
```bash
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
```

These commands reserve gateway and infrastructure IP addresses.

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

Each DHCP pool ensures hosts receive IP addressing appropriate to their VLAN.

<img src="https://github.com/user-attachments/assets/3b57ad04-99f8-4bc3-9791-5527d4d7ecf3" />
<img src="https://github.com/user-attachments/assets/87ac6168-1c4c-4fb1-990e-dc79aa9c7faf" />

---

## Interconnecting Both Routers

### Router A
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

Static routes inform the router how to reach remote VLAN networks.

<img src="https://github.com/user-attachments/assets/90e7470c-b65b-4164-8a65-89e67b0abbb3" />

### Router B
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

Static routing is required on both routers to enable bidirectional communication.

<img src="https://github.com/user-attachments/assets/a7999a33-f2f4-4675-a316-37af9b9db9b6" />

---

## Validation and Testing
Successful DHCP address assignment confirms VLAN and trunk configuration. End-to-end ICMP testing verifies routing correctness.

<img src="https://github.com/user-attachments/assets/cae2ac0d-f13c-4907-9c4e-255d452e60b2" />
<img src="https://github.com/user-attachments/assets/87d457d6-e828-46d2-b92f-af74e2fc513f" />
<img src="https://github.com/user-attachments/assets/e8b66e5a-284f-41d5-83a5-f3d55129230b" />

---

## Conclusion
This lab validates enterprise VLAN segmentation, DHCP automation, router-on-a-stick deployment, and static routing between multiple routers. The final topology confirms full bidirectional connectivity across all VLANs and routed networks.
