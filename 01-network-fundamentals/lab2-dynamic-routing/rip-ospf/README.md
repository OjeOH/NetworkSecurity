# 🔄 Lab 2 — Dynamic Routing: RIP v2 & OSPF

**Course:** INFO8491 — Network Fundamentals | Conestoga College, Winter 2024  
**Submitted:** March 2, 2024  
**Tools:** Cisco Packet Tracer, Cisco 2811 Routers (×3), Cisco 2960 Switch

---

## Objective

Replace static routing with dynamic routing protocols — first **RIP version 2**, then **OSPF** — on a three-router topology representing interconnected cities (Waterloo, Kitchener, Stratford). This lab demonstrates how dynamic routing automates path discovery and adapts to network changes without manual intervention.

---

## Topology

```
[PC-Waterloo]──[SW1]──[R-Waterloo]──[R-Kitchener]──[Server-Kitchener]
                                           │
                                     [R-Stratford]──[Server-Stratford]
                                           │
                                       [PC2 (Kitchener)]
```

**Devices used:**
- 3× Cisco 2811 Routers (R-Waterloo, R-Kitchener, R-Stratford)
- 1× Cisco 2960 Switch
- 2× PC endpoints
- 2× Servers

---

## Part A — RIP Version 2

### Why RIP over Static Routing?

In a growing network, maintaining static routes on every router becomes operationally unsustainable. RIP v2 eliminates this by:

- **Automatically advertising** connected networks to neighboring routers
- **Adapting dynamically** when links go up or down
- **Propagating changes** through the network without admin intervention

RIP v2 supports classless routing (CIDR / VLSM) and sends subnet mask information with route updates — a major improvement over RIP v1.

### Configuration Summary

```cisco
! On each router:
router rip
 version 2
 no auto-summary
 network <directly-connected-network>
```

`no auto-summary` is critical — it prevents RIP from summarizing routes to classful boundaries, which would break connectivity on discontiguous networks.

### Verification Commands

```cisco
show ip protocols          ! Confirms RIP is running, timers, networks advertised
show ip route              ! Shows routes learned via RIP (marked 'R')
show ip rip database       ! RIP route database
debug ip rip               ! Real-time view of RIP update packets
```

Debug output confirmed that RIP updates were being sent and received between all three routers every 30 seconds (default update timer).

---

## Part B — OSPF (Open Shortest Path First)

### Why OSPF over RIP?

OSPF is the industry-standard IGP for enterprise networks. Key advantages over RIP:

| Feature | RIP v2 | OSPF |
|---|---|---|
| Metric | Hop count (max 15) | Cost (based on bandwidth) |
| Convergence | Slow (~90–180 seconds) | Fast (sub-second with tuning) |
| Scalability | Small networks only | Scales to thousands of routers |
| Algorithm | Bellman-Ford (distance vector) | Dijkstra SPF (link-state) |
| Updates | Periodic (every 30s) | Event-triggered only |

### Configuration Summary

```cisco
! On each router:
router ospf 1
 network <network> <wildcard-mask> area 0
```

All routers were placed in **Area 0** (the backbone area), as this is a single-area deployment. This is the most common starting point for enterprise OSPF deployments.

### Verification Commands

```cisco
show ip ospf neighbor        ! Confirms adjacency formation (FULL state)
show ip ospf database        ! Link-state database (LSAs)
show ip ospf interface       ! OSPF parameters per interface
show ip route ospf           ! Routes learned via OSPF (marked 'O')
```

---

## End-to-End Connectivity Test

From PC-Waterloo, successfully:
- Reached the **Kitchener server's web interface** via HTTP
- Reached the **Stratford server's web interface** via HTTP
- Pinged all remote hosts across both routing protocol configurations

---

## Key Takeaways

> **Real-world relevance:** Every ISP, enterprise data center, and campus network runs a dynamic routing protocol. Understanding when to use RIP vs. OSPF — and how to verify and troubleshoot routing tables — is a core skill for any network engineer or security professional.

- RIP's 15-hop limit makes it unsuitable for large networks; OSPF has no such limit
- OSPF's use of bandwidth as a metric produces better path selection than RIP's hop count
- `no auto-summary` is almost always required in modern RIP deployments
- Debug mode is an invaluable real-time troubleshooting tool but should be used cautiously in production

---

## Files

| File | Description |
|---|---|
| `Ooje7862-INFO8491-24W-Portfolio_2_RIP_.pkt` | Packet Tracer file — RIP v2 configuration |
| `Ooje7862-INFO8491-24W-Portfolio_2_Ospf_.pkt` | Packet Tracer file — OSPF configuration |
| `OOJE7862_INFO8491-24W-Portfolio_2__RIP___OSPF___1_.docx` | Full lab report with screenshots and analysis |
