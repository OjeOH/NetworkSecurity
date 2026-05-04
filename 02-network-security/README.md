# 🛡️ Course 2 — Network Security II (INFO8501)
**Conestoga College | Summer 2024 | Prof. Tariq Mahood**

---

## Overview

This course shifts from network design into network **defense and attack**. Labs progress from firewall fundamentals through to hands-on penetration testing, VPN deployment, and intrusion detection — building the skill set of a security-focused network engineer or junior SOC analyst.

The primary platform is **Palo Alto Networks NGFW (PAN-OS 10.x)** running on physical lab kits via **VMware vSphere**, complemented by real Linux VMs running CentOS 7.

---

## The Story of This Module

The labs in this course tell a sequential story:

1. **Build the perimeter** — deploy and configure a Palo Alto NGFW with proper zone segmentation and a PKI infrastructure
2. **Understand the threat** — attack an FTP server and capture credentials with Wireshark to understand *why* cleartext protocols are dangerous
3. **Secure the perimeter** — implement Site-to-Site IPsec VPN to protect data in transit between branch offices
4. **Enable secure remote access** — deploy GlobalProtect VPN so remote workers can connect safely
5. **Watch for attackers** — deploy SNORT IDS to detect malicious traffic on the wire
6. **Prove it all works** — demonstrate comprehensive Palo Alto configuration skills under exam conditions

---

## Labs in This Module

| Lab | Focus | Key Technology |
|---|---|---|
| [Lab 1 — Firewall & PKI](./lab1-firewall-pki/) | Zone segmentation, Certificate Authority | Palo Alto PAN-OS, vSphere |
| [Lab 2 — Penetration Testing](./lab2-penetration-testing/) | FTP exploit, Wireshark sniffing, RSA SSH auth | Wireshark, CentOS, PuTTY |
| [Lab 3 — IPsec VPN](./lab3-ipsec-vpn/) | Site-to-site VPN, IKE Phase 1 & 2 | Palo Alto NGFW |
| [Lab 4 — GlobalProtect VPN](./lab4-globalprotect-vpn/) | Remote access VPN, SSL/TLS profiles | Palo Alto GlobalProtect |
| [Lab 5 — SNORT IDS](./lab5-snort-ids/) | Intrusion detection, rule management | SNORT, PulledPork, CentOS 7 |
| [Final Exam](./final-exam/) | Full Palo Alto config + written security theory | Palo Alto PAN-OS 10.x |

---

## Environment

- **Firewall:** Palo Alto PA-series (Kit 085 / Kit 086 / Kit 088) — PAN-OS 10.0/10.1
- **Hypervisor:** VMware vSphere
- **OS:** CentOS 7, Windows Server 2019
- **Network Analyzer:** Wireshark
- **IDS:** SNORT 3 + PulledPork

---

## Key Concepts Covered

- **Next-Generation Firewall (NGFW)** — application-aware policy enforcement beyond port/protocol
- **Network Segmentation** — Inside / DMZ / Guest / Internet zones
- **PKI** — Certificate Authority creation, private key management, certificate signing
- **IPsec** — Security Associations, IKE Phase 1 (ISAKMP), IKE Phase 2 (IPsec SA)
- **VPN** — Site-to-site and remote access (GlobalProtect) architectures
- **Penetration Testing** — FTP credential interception, Wireshark analysis
- **RSA Authentication** — Public/private key pairs for passwordless SSH
- **IDS/IPS** — SNORT rule sets, PulledPork automatic rule updates, traffic analysis
