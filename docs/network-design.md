# Network Design Notes

## Site Layout

Four sites connected through a shared ISP router. Each site has:
- **ASA 5506-X** — edge firewall; terminates IPsec tunnels; handles NAT, ACLs, and zone security
- **ISR 4331** — interior router; handles per-site VLAN routing and OSPF

The ASAs hold the public IPs (11.0.0.x), which is why VPN tunnels terminate on the ASAs rather than the ISRs.

## Routing

OSPF Area 0 runs between all 4 sites via the ISP router. Each ASA redistributes connected routes into OSPF so that all interior subnets are reachable across the WAN.

## IPsec VPN Design

**Why full mesh?** A hub-and-spoke design routes site-to-site traffic through HQ, adding latency and creating a single point of failure. Full mesh gives every site a direct encrypted path to every other site.

**Tunnel pairs (6 total):**
| Tunnel | Peer A | Peer B |
|--------|--------|--------|
| 1 | EMAGINE-FW (11.0.0.42) | ORLANDO-FW (11.0.0.44) |
| 2 | EMAGINE-FW (11.0.0.42) | DC-FW (11.0.0.46) |
| 3 | EMAGINE-FW (11.0.0.42) | CHICAGO-FW (11.0.0.48) |
| 4 | ORLANDO-FW (11.0.0.44) | DC-FW (11.0.0.46) |
| 5 | ORLANDO-FW (11.0.0.44) | CHICAGO-FW (11.0.0.48) |
| 6 | DC-FW (11.0.0.46) | CHICAGO-FW (11.0.0.48) |

Each ASA has 3 crypto map entries (one per remote peer). The interesting-traffic ACLs use `permit ip any any` scoped per-peer to keep the config readable.

**IKEv1 parameters:**
- Encryption: 3DES
- Hash: SHA-1
- Auth: pre-shared key (`cisco`)
- DH group: 2
- Lifetime: 86400 seconds

Note: IKEv2 is not available on ASAs in Cisco Packet Tracer — IKEv1 is the only option in the simulator.

## Dual ISP Design

**Why floating static routes over IP SLA?** IP SLA adds complexity (probe configuration, track objects, conditional routes) without meaningful benefit in a lab environment. Floating static routes achieve the same failover outcome with a simpler config that's easier to read and verify.

Each ASA has two default routes:
```
route outside 0.0.0.0 0.0.0.0 11.0.0.254 1    ! AD 1  — always preferred
route isp2    0.0.0.0 0.0.0.0 12.0.0.254 10   ! AD 10 — used only when ISP1 is down
```

When the `outside` interface goes down, its route is removed from the routing table and the `isp2` route with AD 10 becomes the active default.

**ISP2 interface IPs:**
| Site | G1/3 IP |
|------|---------|
| EMAGINE-FW | 12.0.0.42 |
| ORLANDO-FW | 12.0.0.44 |
| DC-FW | 12.0.0.46 |
| CHICAGO-FW | 12.0.0.48 |

Note: the ISP2 router device is not currently present in the PT topology — G1/3 is configured and routes exist but the far-end router is absent. Adding it is a pending enhancement.

## Security Policy

Each ASA enforces:
- **Security levels**: inside = 100, outside = 0, isp2 = 0
- **Inspection policy**: stateful TCP/UDP/ICMP inspection on inside-to-outside traffic
- **Outside ACL**: permits ICMP replies, TCP 80/443, and UDP 500 (IKE) inbound; denies all else

The outside ACL must explicitly permit UDP 500 — in Packet Tracer, the ACL is applied to all inbound traffic including traffic destined for the ASA itself (IKE exchange). Without it, Phase 1 never starts.
