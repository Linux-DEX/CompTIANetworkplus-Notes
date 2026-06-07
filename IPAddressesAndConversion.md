---
title: CompTIA Network+
description: IP Addresses and Conversion
author: Linux-DEX
options: toc:3
caption: My Image
---

# TABLE OF CONTENT
- [TCP and IP](#tcp-and-ip)
- [UDP](#udp)
- [ARP & RARP](#arp--rarp)
- [ICMP - Internet Control Message Protocol](#icmp---internet-control-message-protocol)
- [IGMP - Internet Group Management Protocol](#igmp---internet-group-management-protocol)

# TCP and IP
**TCP(Transmission Control Protocol)** : Connection oriented, guaranteed delivery, with reliability features that includes:
- Flow control - for reduces overloading
- Checksum mechanism (minor)
- Limits maximum segment size (prevent MTU/MUT mismatch) For application that depend on reliable data delivery

**IP(Internet Protoco)** : Connectionless, best effort delivery. Outlines:
- Information structure.
- Navigation and route establishment.
- Specifications to connect to the Internet.

# UDP
**UDP(User datagram protocol)** : Connectionless protocol for those applications that require expedience over reliability features:
- Best effort delivery.
- Stateless protocol prefers packet loss over delay in retransmitting.
- Checksum for data integrity.
- Addresses port numbers for functions: (DNS:53).
- VoIP or on-line gaming.

# ARP & RARP
- ARP - Address Resolution Protocol
- RARP - Reverse Address Resolution Protocol

**ARP** : Request and replay protocol that:
- Maps IP addresses to MAC addresses.

**ARP Process**:
1. Receives IP address from IP.
2. Looks in ARP Tables for mapping.

2a. If there:
    3. MAC returned to IP

2b. If not:
    3. Broadcast message sent.
    4. Target responds(Unicast).
    5. MAC Added to table.
    6. MAC returned to IP.
 
RARP Maps MAC addresses to IP addresses.

# ICMP - Internet Control Message Protocol
- Attempts to report on system status.
- Used for diagnostic and testing purposes.
- Utilities from ICMP: Tracert, Pathping, Ping.
- Internet Layer Protocol.
- Ping uses ICMP echo requests.

# IGMP - Internet Group Management Protocol
- Used to establish memberships for multicast groups.










7:50
