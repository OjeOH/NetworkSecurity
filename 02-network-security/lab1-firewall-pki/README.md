# 🔥 Lab 1 — Palo Alto NGFW: Zone Segmentation & PKI

**Course:** INFO8501 — Network Security II | Conestoga College, Summer 2024  
**Submitted:** May 24, 2024  
**Tools:** Palo Alto PAN-OS 10.1, VMware vSphere, CentOS/Linux VMs

---

## Objective

Deploy a **Palo Alto Next-Generation Firewall** from scratch in a virtual lab environment — building a segmented network with clearly defined security zones, then establishing a **Private Certificate Authority (CA)** and signing digital certificates for use in secure communications.

---

## Network Topology

```
Internet (Untrust Zone)
        │
   [Palo Alto NGFW]
   /      |      \
Inside   DMZ    Guest
 Zone    Zone    Zone
  │       │       │
[VMs]  [Servers] [Guest VMs]
```

**Firewall Interfaces:**
- `Ethernet1/1` → **Internet** (Untrust) — `172.17.1.0/24`
- `Ethernet1/2` → **Inside** (Trust) — `172.17.200.0/24`
- `Ethernet1/3` → **DMZ** — `172.17.60.0/24`
- `Ethernet1/4` → **Guest** — `172.17.70.0/24`

---

## Part 1 — Zone-Based Firewall Configuration

### Why Zones Matter

Traditional firewalls used IP addresses and ports to make allow/deny decisions. A Next-Generation Firewall using **zone-based policy** adds a layer of logical segmentation:

- **Traffic between zones** is explicitly evaluated against security policies
- **Traffic within a zone** is implicitly trusted (or can be restricted further)
- Zones map to business intent: "DMZ servers should be reachable from the Internet but not from the Guest network"

### Steps Performed

1. **Logged into Palo Alto GUI** via access server at `10.173.254.106`
2. **Loaded base config** to reset the firewall to a known clean state
3. **Created four zones:** Inside, Internet, DMZ, Guest
4. **Assigned IP addresses** to each firewall interface
5. **Bound interfaces to zones** — each physical/virtual interface belongs to exactly one zone
6. **Created a virtual router** and assigned all interfaces — enabling the firewall to route between zones
7. **Built inter-zone security policies** to control which traffic flows are permitted

### Zone Configuration (PAN-OS CLI equivalent)

```bash
# Zone creation (done via GUI — Network > Zones)
set zone inside network layer3 [ ethernet1/2 ]
set zone dmz    network layer3 [ ethernet1/3 ]
set zone guest  network layer3 [ ethernet1/4 ]
set zone internet network layer3 [ ethernet1/1 ]
```

---

## Part 2 — Certificate Authority (PKI) Deployment

### Why a Private CA?

A **Private Certificate Authority** is used in enterprise environments to:
- Issue certificates to internal servers, VPNs, and user devices
- Establish encrypted, authenticated communication inside the organization
- Avoid the cost of public CA certificates for internal services

In this lab, I built the full PKI chain: root key → CA → signed certificates.

### PKI Architecture

```
[Root Private Key]
       │
       ▼
[Certificate Authority (CA)]
       │
       ├──► Signs → [Server Certificate]
       └──► Signs → [VPN Certificate]
```

### Steps Performed

**Step 1 — Generate the Root Private Key**

```bash
openssl genrsa -out ITNSCA.key 2048
```

> **Troubleshooting note:** The initial command used `.cert` extension which caused an "access denied" error. Resolved by using `.crt` — the correct X.509 certificate extension.

**Step 2 — Create the Certificate Authority**

```bash
openssl req -new -x509 -key ITNSCA.key -out ITNSCA.crt -days 3650
```

The CA is now ready to sign client and server certificates.

**Step 3 — Sign Certificates with the CA**

Generated certificate signing requests (CSRs) and signed them using the private CA, producing trusted certificates for use in VPN and HTTPS.

---

## Security Concepts Demonstrated

| Concept | Implementation |
|---|---|
| Defense in depth | Multiple security zones with explicit policies between each |
| Principle of least privilege | Guest zone cannot reach Inside zone |
| DMZ isolation | Internet-facing services in DMZ, isolated from Inside |
| PKI trust chain | Private CA → signed certs eliminate need for self-signed |
| Certificate lifecycle | Generated, signed, and ready for deployment |

---

## Key Takeaways

> **Real-world relevance:** Every enterprise network uses zone-based segmentation. A breach in the Guest Wi-Fi zone should *never* be able to reach finance servers in the Inside zone — this is enforced at the firewall. Similarly, internal PKI is a foundational building block for SSL inspection, VPN authentication, and zero-trust architectures.

- Zone-based firewalls provide more meaningful, business-aligned security policies than ACL-based filtering
- A private CA gives full control over certificate issuance and revocation — essential for VPN client certificates
- The Palo Alto commit model (stage changes then commit atomically) prevents partial configuration states
- All changes must be committed before they take effect — this prevents partial misconfigurations

---

## Files

| File | Description |
|---|---|
| `Ooje7862-INFO8501-24S-LAB1__1_.docx` | Full lab report — topology, CA deployment, screenshots |
