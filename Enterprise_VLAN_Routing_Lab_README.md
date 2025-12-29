
# Enterprise VLAN Routing Lab with Multi-Router Static Routing

## Project Overview
This lab demonstrates enterprise-style network segmentation and connectivity using VLANs, router-on-a-stick, DHCP, and multi-router static routing in Cisco Packet Tracer. Two independent routed environments are built first (each with two VLANs), then interconnected through a dedicated point-to-point /30 link so hosts in any VLAN can communicate end-to-end across both routing domains.

---

## Why These IP Ranges Were Chosen
### VLAN Subnets (/24)
Each VLAN uses a /24 subnet (255.255.255.0) because it is simple, scalable, and standard for enterprise access networks. It supports up to 254 hosts per VLAN and makes troubleshooting and documentation easier.

### Router-to-Router Link (/30)
The inter-router link uses a /30 subnet (255.255.255.252) to conserve address space while supporting exactly two router interfaces, which is best practice for point-to-point links.

---

## Lab Objectives
- Implement VLAN segmentation on switches
- Configure router-on-a-stick with 802.1Q subinterfaces
- Provide DHCP services per VLAN
- Connect two independent routed networks
- Enable full bidirectional communication using static routing
- Validate end-to-end connectivity across all VLANs

---

## Conclusion
This project demonstrates enterprise networking fundamentals including VLAN design, DHCP automation, router-on-a-stick configuration, and static routing between routers. Successful validation confirms full connectivity across all VLANs and routing domains.
