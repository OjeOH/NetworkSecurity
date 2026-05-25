# 🌍 Lab 4 — Remote Access VPN with Palo Alto GlobalProtect

Network Security II  
**Tools:** Palo Alto PAN-OS 10.1 (Kit 085 / Kit 088), VMware vSphere

---

## Objective

Enable **secure remote access** for users connecting from outside the corporate network using **Palo Alto GlobalProtect** — the enterprise remote-access VPN solution built into the Palo Alto NGFW platform. Remote users authenticate, receive an IP address from the VPN pool, and are tunneled into the organization's network as if they were physically on-site.

---

## Architecture

```
[Remote User / Internet]
         │
         │ HTTPS/SSL (GlobalProtect Agent)
         ▼
[Palo Alto NGFW — GlobalProtect Portal]
         │ (authenticates user, pushes config)
         ▼
[Palo Alto NGFW — GlobalProtect Gateway]
         │ (terminates VPN tunnel)
         ├──► Inside Zone  (172.16.10.0/24)
         ├──► DMZ Zone     (172.16.200.0/24)
         └──► Tunnel Zone  (VPN users)
```

**Network Addressing:**
- Internet interface: `172.16.1.0/24`
- Inside network: `172.16.10.0/24`
- DMZ network: `172.16.200.0/24`

---

## GlobalProtect Architecture: Portal vs. Gateway

GlobalProtect uses two components on the firewall:

| Component | Role |
|---|---|
| **GP Portal** | First point of contact for the agent. Authenticates the user and distributes the gateway configuration. |
| **GP Gateway** | Terminates the VPN tunnel. Enforces security policy for remote users. Can apply HIP (Host Information Profile) checks. |

In small deployments, the portal and gateway can run on the same firewall interface. In enterprise deployments, they are often separated for scalability and redundancy.

---

## Configuration Steps

### Step 1 — Interface & Zone Setup

Configured firewall interfaces and zones as in Lab 1, adding a **VPN/Tunnel zone** for remote access users:

```
Network > Interfaces > Tunnel > Add
 - Interface: tunnel.1
 - Security Zone: VPN-Users
```

### Step 2 — Root Certificate Authority

Generated a self-signed root CA on the firewall to sign all VPN certificates:

```
Device > Certificate Management > Certificates
 - Name: BCsite-Root-CA
 - Common Name: BCsite Root CA
 - ✓ Certificate Authority
```

### Step 3 — SSL/TLS Server Profiles

Created SSL/TLS profiles for both the Portal and the Gateway, binding each to a certificate signed by the root CA:

- **Portal SSL Profile** → Portal certificate (signed by Root CA)
- **Gateway SSL Profile** → Gateway certificate (signed by Root CA)

### Step 4 — User Authentication

Created a local user account and configured a **Local Database authentication profile**:

```
Device > Local User Database > Users
 - Username: vpnuser
 - Password: <strong-password>

Device > Authentication Profile
 - Type: Local Database
 - Users: vpnuser
```

### Step 5 — GlobalProtect Portal Configuration

```
Network > GlobalProtect > Portals > Add
 - Interface: Ethernet1/1 (Internet-facing)
 - Authentication: <Auth-Profile>
 - SSL/TLS Profile: Portal-SSL
 - Gateway: <GW-IP>
```

### Step 6 — GlobalProtect Gateway Configuration

```
Network > GlobalProtect > Gateways > Add
 - Interface: Ethernet1/1
 - Tunnel Interface: tunnel.1
 - Authentication: <Auth-Profile>
 - SSL/TLS Profile: Gateway-SSL
 - IP Pool: 10.10.10.0/24  (addresses assigned to VPN clients)
 - Split Tunnel: Disabled (all traffic goes through VPN — "full tunnel")
```

### Step 7 — Security Policies

Created policies to permit traffic from the VPN-Users zone to Inside and DMZ zones:

```
Source Zone:      VPN-Users
Destination Zone: Inside
Action:           Allow
```

---

## Testing the VPN

Connected from a machine on the DMZ network (simulating an external user) to the GlobalProtect portal:

1. Opened browser → `https://172.16.1.x` (portal IP)
2. Downloaded and installed the GlobalProtect agent
3. Authenticated with the local user account
4. Tunnel established — received IP from the VPN pool
5. ✅ Successfully accessed resources on the Inside network (`172.16.10.0/24`)

---

## Key Security Benefits of GlobalProtect

| Feature | Benefit |
|---|---|
| Certificate-based portal/gateway | Prevents man-in-the-middle attacks |
| HIP (Host Information Profile) | Can enforce endpoint compliance before granting access |
| Full-tunnel mode | All traffic (including Internet) goes through corporate firewall for inspection |
| Split-tunnel mode | Only corporate traffic tunneled — reduces firewall load |
| MFA integration | Can be paired with RADIUS/SAML for multi-factor auth |

---

## Key Takeaways

> **Real-world relevance:** Since 2020, remote access VPN has become critical infrastructure. GlobalProtect is deployed by thousands of enterprises worldwide. Understanding how to configure the portal, gateway, certificate trust, authentication profiles, and IP pools — and how to troubleshoot when a client can't connect — is a highly marketable skill for any network security role.

- The portal and gateway certificates **must** be trusted by the client — either through a public CA or by distributing the root CA certificate to endpoints
- IP pool exhaustion is a real operational issue in large deployments — right-size the pool
- HIP checks (checking if the client has AV, disk encryption, OS patches) transform a VPN from "connectivity" to "zero-trust access"
- Security policy must explicitly permit VPN-Users zone traffic — implicit deny catches many deployment errors

---

## Files

| File | Description |
|---|---|
| `Portfolio3_Ooje7862__1_.xml` | Palo Alto XML config export — GlobalProtect VPN configuration |
| `Ooje7862-INFO8501-24S-SEC1-PORTFOLIO_3__3_.docx` | Full lab report with topology, screenshots, and analysis |
