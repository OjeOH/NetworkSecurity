# 🔐 Lab 3 — Site-to-Site IPsec VPN (Palo Alto NGFW)

**Course:** INFO8501 — Network Security II | Conestoga College, Summer 2024  
**Tools:** Palo Alto PAN-OS 10.1 (Kit 086), Cisco Router (BCsite1), VMware vSphere

---

## Objective

Design, configure, and validate a **Site-to-Site IPsec VPN tunnel** between two geographically separated office sites — **BCsite1** (my configuration) and **Toronto Site1** (partner's configuration) — using Palo Alto NGFW as the VPN gateway, with a Palo Alto NGFW (NGFW) and Cisco router as the on-premises infrastructure.

This lab simulates a real enterprise scenario: two branch offices that need to securely share internal resources across the public Internet without their data being readable to anyone on the path.

---

## Network Design

### BCsite1 Topology (My Configuration)

```
[Internet/ISP]
      │
      │ Ethernet1/1 — 172.17.1.0/24
      │
 [Palo Alto NGFW — BCsite1]
  │           │           │
Eth1/2       Eth1/3     Eth1/4
Inside       DMZ         Guest
172.17.200   172.17.60   172.17.70
.0/24        .0/24       .0/24
  │
[Cisco BCsite1 Router]
```

### VPN Tunnel

```
BCsite1 NGFW  ──────[IPsec Tunnel]──────  Toronto Site1 NGFW
172.17.1.x                               (Partner's IP)
     │                                          │
 172.17.200.0/24                        Internal Network
   (Inside)                               (Toronto)
```

---

## IPsec Concepts

### Security Association (SA)

An **IPsec Security Association** is a one-way logical relationship between two peers that defines:
- Which encryption algorithm to use (e.g., AES-256)
- Which authentication algorithm to use (e.g., SHA-256)
- The lifetime of the keys
- The traffic that is protected (via the crypto ACL / proxy ID)

Because SA is unidirectional, a VPN tunnel requires **two SAs** — one in each direction.

### IKE Phase 1 — Establishing the Secure Channel

IKE Phase 1 creates the **ISAKMP SA** — a secure, authenticated management channel between the two VPN peers. This channel is used to negotiate Phase 2.

Phase 1 parameters (must match on both sides):
- **Encryption:** AES-256 (or 3DES for legacy)
- **Hashing/Integrity:** SHA-256
- **DH Group:** Group 14 (2048-bit) or higher
- **Authentication:** Pre-shared key (PSK) or certificate
- **Lifetime:** 28800 seconds (8 hours) typical

### IKE Phase 2 — Establishing the Data Tunnel

IKE Phase 2 negotiates the **IPsec SA** — the actual tunnel that carries user data.

Phase 2 parameters (must match on both sides):
- **Protocol:** ESP (Encapsulating Security Payload)
- **Encryption:** AES-256
- **Hashing:** SHA-256
- **PFS (Perfect Forward Secrecy):** Enabled (DH Group 14)
- **Proxy IDs:** Define which subnets are protected by the tunnel

```
Phase 1 (ISAKMP SA) — secures the negotiation
        │
        ▼
Phase 2 (IPsec SA) — secures the actual data
```

---

## Configuration Steps (BCsite1 — Palo Alto)

### Step 1 — Interface Assignment

Assigned each Palo Alto interface to its intended zone and configured IP addresses:

| Interface | Zone | IP Address |
|---|---|---|
| Ethernet1/1 | Internet (Untrust) | 172.17.1.x/24 |
| Ethernet1/2 | Inside (Trust) | 172.17.200.x/24 |
| Ethernet1/3 | DMZ | 172.17.60.x/24 |
| Ethernet1/4 | Guest | 172.17.70.x/24 |

### Step 2 — Tunnel Interface

Created a dedicated **tunnel interface** for the VPN — this logical interface carries IPsec traffic and is assigned to a VPN zone:

```
Network > Interfaces > Tunnel > Add
 - Tunnel Interface: tunnel.1
 - Zone: VPN
 - IP: (optional, used for monitoring)
```

### Step 3 — IKE Gateway Configuration

```
Network > Network Profiles > IKE Gateways
 - Interface: Ethernet1/1 (Internet-facing)
 - Peer IP: <Toronto-Site1-Public-IP>
 - Pre-Shared Key: <shared-secret> (agreed with partner)
 - IKE Version: IKEv1
 - Exchange Mode: Main Mode
```

### Step 4 — IPsec Tunnel Configuration

```
Network > IPsec Tunnels
 - Tunnel Interface: tunnel.1
 - IKE Gateway: <BCsite1-IKE-GW>
 - IPsec Crypto Profile: AES-256 / SHA-256
 - Proxy IDs:
    Local: 172.17.200.0/24
    Remote: <Toronto-Inside-Network>
```

### Step 5 — Security Policy

Created a security policy permitting traffic from the Inside zone to flow through the tunnel to the VPN zone (and return traffic in the reverse direction).

### Step 6 — Static Route

Added a static route pointing Toronto's internal subnet toward `tunnel.1`:

```
Network > Virtual Routers > Static Routes
 - Destination: <Toronto-Inside-Network>
 - Next Hop: tunnel.1
```

---

## Verification

The tunnel status was verified in:
- **Network > IPsec Tunnels** — green status indicators for Phase 1 and Phase 2
- **Monitor > Logs > System** — IKE negotiation success messages
- **Ping test** — pinging hosts on Toronto's Inside network from BCsite1's Inside network through the tunnel

---

## Key Takeaways

> **Real-world relevance:** Site-to-site VPNs are the backbone of enterprise WAN connectivity. Any organization with multiple offices, cloud connections (AWS Direct Connect, Azure VPN Gateway), or partner integrations relies on IPsec VPN. Understanding IKE Phase 1 and Phase 2 — and the parameters that must match — is essential for any network or security engineer.

- Phase 1 and Phase 2 parameters **must** match exactly on both peers — even one mismatched setting will prevent tunnel establishment
- Pre-shared keys are convenient but certificate-based authentication is more scalable and secure at enterprise scale
- PFS (Perfect Forward Secrecy) ensures that compromise of one session key doesn't expose past or future sessions
- Always verify both Phase 1 (ISAKMP SA) and Phase 2 (IPsec SA) are `UP` before testing connectivity

---

## Files

| File | Description |
|---|---|
| `Ooje7862-INFO8501-24S-Sec1-Firewall_Config` | Palo Alto XML configuration export — BCsite1 NGFW |
| `Ooje7862-INF08501-Sec1-24S-Portfolio2__2_.docx` | Lab report — IPsec VPN section with topology and screenshots |
