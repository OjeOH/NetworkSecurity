# 🔐 Lab 2 — EIGRP Routing with MD5 Authentication

Network Fundamentals 
March 2, 2024  
**Tools:** Cisco Packet Tracer, Cisco 2811 Routers (×6), Cisco 2960 Switches (×6)

---

## Objective

Deploy **EIGRP (Enhanced Interior Gateway Routing Protocol)** across a six-router hub-and-spoke topology — and then secure the routing protocol itself using **MD5 authentication**, preventing rogue routers from injecting false routing information into the network.

---

## Topology — Hub-and-Spoke

```
              [Hub Router]
             /    |    |   \
            /     |    |    \
      [Spoke1] [Spoke2] [Spoke3] [Spoke4]
         |        |        |        |
        SW1      SW2      SW3      SW4
         |        |        |        |
        PC1      PC2      PC3      PC4–PC6
```

**Devices used:**
- 6× Cisco 2811 Routers (1 Hub + 4 Spokes, with additional connections)
- 6× Cisco 2960 Switches
- 6× PC endpoints

---

## Part A — EIGRP Configuration

### Why EIGRP?

EIGRP is Cisco's advanced distance-vector protocol. Compared to RIP and OSPF:

| Feature | RIP v2 | OSPF | EIGRP |
|---|---|---|---|
| Algorithm | Bellman-Ford | Dijkstra SPF | DUAL (Diffusing Update Algorithm) |
| Convergence | Slow | Fast | Very Fast |
| Metric | Hop count | Cost (bandwidth) | Composite (bandwidth + delay) |
| Partial updates | No | No | Yes — only sends changes |
| Cisco proprietary | No | No | Originally yes (now open standard) |

EIGRP's DUAL algorithm ensures **loop-free routing** at all times and provides an instant failover path (the *feasible successor*) when a primary route fails.

### Loopback Interfaces

Loopback interfaces were configured on each spoke router to simulate additional networks (e.g., representing remote branch offices or management networks). Loopbacks are always `UP/UP` — they never go down unless explicitly shut, making them ideal for testing and stable route sources.

```cisco
interface loopback 0
 ip address <loopback-ip> 255.255.255.0
```

### Passive Interfaces

Spoke routers were configured with **passive interfaces** on their LAN-facing ports. A passive interface still *advertises* the network into EIGRP but does *not* send or receive EIGRP Hello packets on that interface — preventing unnecessary EIGRP neighbor formation with end devices.

```cisco
router eigrp 100
 passive-interface GigabitEthernet0/0
 network <network> <wildcard>
```

### Verification Commands

```cisco
show ip eigrp neighbors          ! Adjacency table — confirms neighbor relationships
show ip eigrp topology           ! Full topology table including feasible successors
show ip protocols                ! Confirms EIGRP is running and AS number
show ip route eigrp              ! Routes learned via EIGRP (marked 'D')
debug eigrp packets              ! Real-time EIGRP packet exchange
```

---

## Part B — EIGRP MD5 Authentication

### Why Authenticate Routing Protocols?

Without route authentication, an attacker who gains access to a network segment can plug in a rogue router and:
- **Inject false routes** to redirect traffic through attacker-controlled paths
- **Cause a black hole** by advertising routes with lower metrics
- **Perform a man-in-the-middle attack** on traffic between sites

MD5 authentication on EIGRP ensures that only routers that possess the correct **key** can form adjacencies and exchange routing updates.

### Keychain Configuration

```cisco
! Step 1 — Define the keychain on each router
key chain EIGRP-AUTH
 key 1
  key-string <shared-secret>

! Step 2 — Apply authentication to the EIGRP interface
interface <WAN-interface>
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-AUTH
```

The **key number** must match on both ends of a link. The key-string is the shared secret — effectively a password that must be consistent across all routers participating in the authenticated EIGRP domain.

### Verification

After applying MD5 authentication:
- Existing EIGRP adjacencies were **re-established** using the authenticated Hello packets
- `show ip eigrp neighbors` confirmed all neighbors returned to **UP** state
- Removing the key-chain from one router caused the neighbor relationship to **drop**, confirming authentication was actively enforced

---

## End-to-End Connectivity Test

From PC1 (Spoke 1), successfully pinged:
- PC2 (Spoke 2), PC3 (Spoke 3), PC4, PC5, PC6 (Spoke 4)

All pings were successful, confirming full inter-spoke reachability through the EIGRP hub topology.

---

## Key Takeaways

> **Real-world relevance:** Routing protocol authentication is a critical security control in any production network. Without it, an attacker with physical access to a network segment (or a compromised router) can manipulate the routing table of the entire network. MD5 keychains are the baseline — modern networks should use SHA-256 HMAC (available on newer IOS versions).

- EIGRP's DUAL algorithm provides instant failover — a key advantage in enterprise WAN designs
- Passive interfaces reduce unnecessary protocol traffic and prevent unintended adjacencies
- MD5 authentication is not encryption — it authenticates the *source* of routing updates but does not encrypt the route data itself
- Key chain numbers must match between neighbors; mismatched key IDs silently break adjacency

---

## Files

| File | Description |
|---|---|
| `Ooje7862-INFO8491-24W-Portfolio_2-_EIGPR_.pkt` | Packet Tracer file — EIGRP + MD5 authentication topology |
| `OOJE7862_INFO8491-24W-Portfolio_2_EIGRP___1_.docx` | Full lab report with screenshots and analysis |
