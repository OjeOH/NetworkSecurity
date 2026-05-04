# 🎓 Final Exam — Comprehensive Palo Alto NGFW Configuration

**Course:** INFO8501 — Network Security II | Conestoga College, Summer 2024  
**Format:** Closed-book practical exam + written theory component  
**Tools:** Palo Alto PAN-OS 10.0/10.1, MS Word

---

## Overview

This final exam tested comprehensive, end-to-end knowledge of network security across two components:

1. **Written component** — Security theory questions covering IPsec, IKE, firewall policy logic
2. **Practical component** — Full Palo Alto NGFW configuration in a live lab environment under exam conditions

The exam was completed individually, with no reference materials, within a strict time limit — demonstrating the ability to work independently and confidently under pressure.

---

## Written Component — Security Theory

### Question 1: Security Association & IKE Phase 1 / Phase 2

**Security Association (SA)**

A Security Association is a one-way agreement between two communicating peers that defines the security parameters for a protected communication channel. It specifies the encryption algorithm, authentication method, session key material, and lifetime for a particular data flow. Because an SA is unidirectional, a typical VPN tunnel requires two SAs — one in each direction.

**IKE Phase 1**

IKE Phase 1 establishes the **ISAKMP SA** — a secure, mutually authenticated management channel between the two VPN endpoints. This phase performs:
- Peer authentication (via pre-shared key or certificates)
- DH key exchange to derive shared keying material
- Negotiation of encryption and integrity algorithms for Phase 1 itself

The result is a protected tunnel used only for Phase 2 negotiation.

**IKE Phase 2**

IKE Phase 2 uses the secure channel from Phase 1 to negotiate the **IPsec SA** — the actual tunnel that carries user data. Phase 2 negotiates:
- ESP encryption algorithm (e.g., AES-256)
- Integrity/authentication algorithm (e.g., SHA-256)
- Traffic selectors (proxy IDs) — which source/destination IP pairs use the tunnel
- Session key lifetime

The Phase 2 SA is what protects user traffic. Phase 2 renegotiates on a shorter interval than Phase 1, providing forward secrecy for data sessions.

---

### Question 2: IKE Transform Set

An **IKE Transform Set** is a named collection of security algorithms and parameters that defines how IKE will protect either the Phase 1 management channel or Phase 2 data sessions. It bundles together:
- **Encryption algorithm** (e.g., AES-128, AES-256, 3DES)
- **Hash/integrity algorithm** (e.g., SHA-1, SHA-256)
- **Authentication method** (pre-shared key or RSA certificates)
- **DH group** (for key exchange — Group 14, Group 19, etc.)

When two VPN peers negotiate, they compare their configured transform sets and select the first mutually supported combination. If no common transform set exists, the VPN cannot be established.

---

### Question 3: Palo Alto Firewall Policy Matching

**Question:** A packet arrives on an interface. It matches the second *and* fourth security policy entries. What happens?

**Answer:** The packet is processed according to the **second policy entry** — the first matching policy wins.

**Explanation:** Palo Alto firewalls (and all stateful firewalls) use a **first-match, top-down policy evaluation** model. The firewall evaluates each security policy rule from top to bottom. The moment the packet matches a rule — regardless of whether other rules further down would also match — that rule's action (Allow, Deny, Drop) is applied and evaluation stops. The fourth policy entry is never reached because the second entry matched first.

This "first-match" model makes **rule ordering critical**: more specific rules must be placed *above* more general rules, or the general rule will absorb traffic that should have hit the specific rule.

---

## Practical Component — NGFW Configuration

Under closed-book exam conditions, configured a Palo Alto NGFW from an uploaded base configuration, performing:

- Interface assignment and zone configuration
- Virtual router setup with default route
- Security policy creation for inter-zone traffic
- Certificate management (self-signed CA + endpoint certificates)
- VPN configuration (IPsec tunnel)
- Policy verification and commit

### Exam Naming Convention

All objects, hostnames, and certificates were named following the required convention:
`HOJ7862-<object-name>`
(Initials: H.O.J., Last 4 digits of student ID: 7862)

---

## Firewall Configuration Export

The complete Palo Alto configuration from the exam is included as an XML export file. This represents a fully functional NGFW configuration including:

- Zone definitions
- Interface configuration
- Security policies
- NAT rules
- Certificate management
- IPsec/VPN configuration

To use: Import via **Device > Setup > Operations > Import named configuration snapshot** in PAN-OS.

---

## Key Takeaways

> This exam validated the ability to configure a complete Palo Alto NGFW deployment from memory — covering zones, policies, routing, PKI, and VPN — within a constrained time window. It represents the culmination of the Network Security II curriculum.

The skills demonstrated are directly applicable to:
- **Network Security Engineer** roles managing NGFW infrastructure
- **SOC Analyst** roles requiring deep understanding of firewall policy logic
- **Network Administrator** roles deploying and maintaining enterprise security perimeters

---

## Files

| File | Description |
|---|---|
| `Ooje7862-FinalExam-INFO8501__1___2_.xml` | Palo Alto NGFW configuration export — Final Exam |
| `Ooje7862NetworkSecurity__2_.xml` | Additional Palo Alto config export |
| `Ooje7862-INFO8501-SEC1-24S-Final-Exam__4_.docx` | Written exam answers — security theory questions |
