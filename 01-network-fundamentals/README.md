# 📡  Network Fundamentals


---

## Overview

This section covers the foundational skills of building, configuring, and securing enterprise networks using Cisco IOS devices. The labs progress from physical topology design through to advanced dynamic routing protocols — reflecting a real-world path that a junior network engineer would follow.

### What I Built

A multi-site network simulation environment using **Cisco 2811 routers** and **2960 switches**, connecting multiple PCs, servers, and laptops across segmented networks. The topology grew from a single LAN into a fully routed inter-network with dynamic routing and authentication.

---

## Labs in This Module

| Lab | Focus | Packet Tracer File |
|---|---|---|
| [Lab 1 — Topology & Hardening](./lab1-topology-hardening/) | Build topology, harden IOS devices, configure SSH & static routes | `OOJE7862-INFO8491-24W-Portfolio-1.pkt` |
| [Lab 2 — RIP & OSPF](./lab2-dynamic-routing/rip-ospf/) | Implement RIP v2 and OSPF on a 3-router topology | `Ooje7862-INFO8491-24W-Portfolio_2_RIP_.pkt` / `Ospf_.pkt` |
| [Lab 2 — EIGRP + MD5](./lab2-dynamic-routing/eigrp-md5/) | Implement EIGRP with MD5 keychain authentication on 6-router hub-and-spoke | `Ooje7862-INFO8491-24W-Portfolio_2-_EIGPR_.pkt` |

---

## Key Concepts Covered

- **Cisco IOS CLI** — navigating privileged/user EXEC and global config modes
- **Network Hardening** — MOTD banners, enable secret passwords, line password enforcement
- **SSH v2** — replacing Telnet with encrypted remote management
- **Static Routing** — manually defining paths between segmented networks
- **Dynamic Routing** — automating path discovery with RIP v2, OSPF, and EIGRP
- **Route Authentication** — securing routing protocol updates with MD5 keychains
- **Debug & Verification** — using `show ip route`, `debug ip rip`, `show ip ospf`, `show ip eigrp neighbors`
