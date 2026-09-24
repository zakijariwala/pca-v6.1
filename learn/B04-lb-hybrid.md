---
block: B04
title: "VPC networking II: load balancing and hybrid"
pillars: [reliability, performance, security]
exam_guide_refs: ["1.3", "2.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/load-balancing/docs/choosing-load-balancer
  - https://docs.cloud.google.com/load-balancing/docs/passthrough-network-load-balancer
  - https://docs.cloud.google.com/cdn/docs/caching
  - https://docs.cloud.google.com/dns/docs/zones/forwarding-zones
  - https://docs.cloud.google.com/network-connectivity/docs/vpn/concepts/overview
  - https://docs.cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview
  - https://docs.cloud.google.com/network-connectivity/docs/interconnect/concepts/partner-overview
  - https://docs.cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview
  - https://docs.cloud.google.com/vpc/docs/private-service-connect
unverified: []
---
# B04 VPC networking II: load balancing and hybrid

## Chunk 1: The load balancer decision
Problem it solves: Google Cloud has many load balancers. A requirements paragraph points to one.

Mental model: Answer four questions in order.

| Question | Options |
|---|---|
| Traffic type | HTTP(S) → Application LB. TCP/UDP/other → Network LB |
| Client location | Internet → external. Inside VPC or hybrid → internal |
| Proxy or passthrough (Network LB) | Proxy terminates the connection. Passthrough keeps client IP and supports UDP, ESP, ICMP |
| Backend spread | Global (multi-region) or regional |

Proxy load balancers open a new connection to the backend. Passthrough load balancers deliver packets unchanged; backends reply straight to clients (direct server return).

Exam signals: "Preserve client source IP" or "UDP" points to passthrough Network LB. "Path-based routing", "single anycast IP worldwide" points to global external Application LB.

Trap: Picking a Network LB for an HTTPS app that needs URL routing.

Pillar tie-in: Reliability and performance.

Check: A game server uses UDP and must see player IPs. Which load balancer family?
<details><summary>Answer</summary>A passthrough Network Load Balancer. It supports UDP and keeps client source IPs.</details>

## Chunk 2: Cloud CDN and Cloud DNS
Problem it solves: Global users need fast static content and name resolution that works across hybrid networks.

Mental model: Cloud CDN caches at Google's edge in front of an external Application Load Balancer. Cache modes: `CACHE_ALL_STATIC` (default), `USE_ORIGIN_HEADERS`, `FORCE_CACHE_ALL`. Cloud DNS hosts public and private zones. Forwarding zones send queries for a domain to on-prem DNS servers; inbound server policies let on-prem resolvers query Cloud DNS.

Exam signals: "Reduce latency for global users of static assets" points to Cloud CDN. "VMs must resolve on-prem hostnames" points to a Cloud DNS forwarding zone.

Trap: `FORCE_CACHE_ALL` on responses with user-specific data. It caches content marked private.

Pillar tie-in: Performance.

Check: VMs in a VPC must resolve `corp.example.internal`, served by on-prem DNS. What do you configure?
<details><summary>Answer</summary>A Cloud DNS forwarding zone for `corp.example.internal` that forwards to the on-prem DNS servers, reachable over VPN or Interconnect.</details>

## Chunk 3: Cloud VPN (HA VPN)
Problem it solves: Encrypted private connectivity to on-prem or another cloud, set up in hours.

Mental model: HA VPN is an IPsec gateway with two interfaces. With tunnels on both interfaces to your peer, it carries a 99.99% availability SLA. Routing is dynamic through Cloud Router with BGP, so the peer device must support BGP. Classic VPN carries a 99.9% SLA. Each tunnel runs over the internet, so throughput per tunnel is limited compared with Interconnect.

Exam signals: "Quick to set up", "encrypted", "modest bandwidth", "99.99%".

Trap: One tunnel on one interface and calling it HA. The 99.99% SLA needs tunnels on both interfaces.

Pillar tie-in: Reliability and security.

Check: What two things does HA VPN need to qualify for the 99.99% SLA?
<details><summary>Answer</summary>Tunnels configured on both HA VPN gateway interfaces, and dynamic routing through Cloud Router with BGP to a peer that supports it.</details>

## Chunk 4: Dedicated vs Partner Interconnect
Problem it solves: High, steady bandwidth to on-prem with predictable latency, off the public internet.

Mental model:

| | Dedicated Interconnect | Partner Interconnect |
|---|---|---|
| Physical link | Your router in a Google colocation facility | Through a service provider |
| Circuit sizes | 10, 100, or 400 Gbps circuits, bundled up to 8 | Provider-dependent |
| VLAN attachment size | 50 Mbps to 400 Gbps | 50 Mbps to 50 Gbps |
| Fits when | You can reach a colocation facility | You can't, or need smaller capacity |

Interconnect isn't encrypted by default. Run HA VPN over Interconnect if the requirement says "encrypted."

Exam signals: "Terabytes per day", "consistent latency", "not over the internet".

Trap: Dedicated Interconnect for a site nowhere near a Google colocation facility.

Pillar tie-in: Performance and reliability.

Check: A company needs 5 Gbps to Google Cloud and has no presence in a colocation facility. Which option?
<details><summary>Answer</summary>Partner Interconnect through a service provider.</details>

## Chunk 5: Network Connectivity Center
Problem it solves: Many VPCs, sites, and clouds. Full-mesh peering and VPNs don't scale.

Mental model: NCC is a global hub with spokes. VPC spokes attach VPC networks. Hybrid spokes attach HA VPN tunnels, Interconnect VLAN attachments, or router appliance VMs. The hub exchanges routes between spokes, giving any-to-any connectivity through one control plane. It's a hub-and-spoke WAN for your whole estate.

Exam signals: "Connect many VPCs and on-prem sites", "transitive connectivity", "multicloud".

Trap: Chaining VPC peerings for transitivity. Peering isn't transitive; NCC is the managed answer.

Pillar tie-in: Operational excellence.

Check: Fifteen VPCs and three data centers need any-to-any routing. What's the managed option?
<details><summary>Answer</summary>A Network Connectivity Center hub with VPC spokes and hybrid spokes.</details>

## Chunk 6: Private Service Connect
Problem it solves: Consume a service in another VPC, or Google APIs, through an internal IP, without peering whole networks.

Mental model: A producer publishes a service behind a service attachment. A consumer creates a PSC endpoint: an internal IP in its own VPC that forwards to that service. Only that one service is exposed, not the producer's network. PSC endpoints can also front Google APIs.

Exam signals: "Expose one service to many consumer VPCs", "overlapping IP ranges between producer and consumer", "private access to Google APIs with an internal IP."

Trap: Peering two whole networks to reach one service, which also fails when their ranges overlap.

Pillar tie-in: Security.

Check: A SaaS provider must serve 200 customer VPCs, some with overlapping ranges. Peering or PSC?
<details><summary>Answer</summary>Private Service Connect. Each customer gets an endpoint in its own VPC; overlapping ranges don't matter and no networks merge.</details>
