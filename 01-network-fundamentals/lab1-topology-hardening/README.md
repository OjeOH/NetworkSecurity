# 🔒 Lab 1 — Network Topology Design & IOS Hardening

**Course:** INFO8491 — Network Fundamentals | Conestoga College, Winter 2024  
**Submitted:** February 6, 2024  
**Tools:** Cisco Packet Tracer, Cisco 2811 Router, Cisco 2960 Switches

---

## Objective

Design a multi-device network topology and progressively harden every network device against unauthorized access — simulating a real-world deployment scenario where a junior network engineer receives brand-new Cisco devices and must configure them from scratch, securely.

---

## Topology

```
[PC1]──┐                          ┌──[PC3]
[PC2]──┤                          ├──[PC4]
       └──[SW1]────[R1]────[SW2]──┘
              │              │
           [Server]       [Laptop]
```

**Devices used:**
- 1× Cisco 2811 Router
- 2× Cisco 2960 Layer 2 Switches
- 4× PC endpoints
- 1× Server
- 1× Laptop

---

## What Was Implemented

### Part 1 — Building the Topology
Connected all devices using correct cable types (straight-through, crossover) and verified physical link status using Packet Tracer.

### Part 2 — MOTD Banner Configuration
Configured a **Message of the Day (MOTD)** login banner on all three devices to warn unauthorized users. This is a legal requirement in many organizations before any access is granted.

```cisco
banner motd # Unauthorized access is prohibited. #
```

### Part 3 — Privileged Mode Password
Set an **enable secret** (MD5-hashed password) to protect privileged EXEC mode on the router and both switches — preventing unauthorized elevation of access.

```cisco
enable secret <password>
service password-encryption
```

### Part 4 — Enforcing Login on Console & VTY Lines
Configured line passwords on all console and VTY (virtual terminal) lines with the `login` command enforced, requiring authentication before any management session.

```cisco
line vty 0 4
 password <password>
 login
line console 0
 password <password>
 login
```

### Part 5 — SSH v2 Configuration
Replaced insecure Telnet access with **SSH version 2**, ensuring all remote management traffic is encrypted. Generated RSA crypto keys and restricted VTY lines to SSH-only.

```cisco
ip domain-name lab.local
crypto key generate rsa modulus 1024
ip ssh version 2
line vty 0 4
 transport input ssh
```

### Part 6 — IP Addressing
Assigned IP addresses to all router interfaces and end devices. Verified connectivity with `ping` between hosts on the same subnet.

### Part 7 — Static Routing (Lab 2 extension)
Configured static routes on the router to enable communication between the two network segments, allowing PC1/PC2 to reach PC3/PC4 and the server.

```cisco
ip route <destination_network> <subnet_mask> <next_hop>
```

---

## Verification

Connectivity was verified by:
- Pinging between all PCs across both subnets
- Successfully SSH-ing from PC workstations into the router
- Confirming that Telnet connections were rejected
- Verifying routing table with `show ip route`

---

## Key Takeaways

> **Why this matters in the real world:** The default configuration on a Cisco device is wide open — no passwords, Telnet enabled, no encryption. Every setting in this lab represents a baseline hardening step that any network engineer must apply before a device is placed in production. Skipping any one of these steps creates a vulnerability that an attacker can exploit.

- Unencrypted Telnet transmits credentials in plaintext — SSH v2 is the minimum acceptable standard
- `enable secret` stores passwords as MD5 hashes; `enable password` stores them in plaintext and should never be used
- MOTD banners are a legal protection for organizations, not just cosmetic
- `service password-encryption` prevents casual reading of plaintext passwords in running configs

---

## Files

| File | Description |
|---|---|
| `OOJE7862-INFO8491-24W-Portfolio-1.pkt` | Packet Tracer topology — Lab 1 (topology + hardening) |
| `OOJE7862-INFO8491-24W-Portfolio-1_2_.pkt` | Packet Tracer topology — Lab 2 (static routing extension) |
| `OOJE7862_INFO8491-24W-Portfolio_1.docx` | Full lab report with screenshots and reflections |
