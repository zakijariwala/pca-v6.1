---
block: P03
title: Network troubleshooting
pillars: [operational-excellence, reliability]
exam_guide_refs: ["4.1", "6.4"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/network-intelligence-center/docs/connectivity-tests/concepts/overview
  - https://docs.cloud.google.com/vpc/docs/flow-logs
  - https://docs.cloud.google.com/firewall/docs/vpc-firewall-rules-logging-overview
  - https://docs.cloud.google.com/load-balancing/docs/firewall-rules
  - https://docs.cloud.google.com/nat/docs/troubleshooting
  - https://docs.cloud.google.com/nat/docs/monitoring
  - https://docs.cloud.google.com/dns/docs/zones/forwarding-zones
  - https://docs.cloud.google.com/vpc/docs/mtu
  - https://cloud.google.com/network-connectivity/docs/vpn/concepts/mtu-considerations
unverified: []
---
# P03 Network troubleshooting

## Chunk 1: The checklist
Problem it solves: "It can't connect" has a dozen causes. Guessing wastes hours.

Mental model: Walk the packet, in this order:
1. Name: does DNS resolve to the address you expect?
2. Route: does a route exist from source to destination?
3. Firewall: does a rule, at every layer, allow the traffic both ways?
4. Egress path: external IP, Cloud NAT, or Private Google Access?
5. Service: is the destination listening and healthy?
6. Size: are large packets dropped (MTU)?

Exam signals: "What should you check first?" The answer tends to be the cheapest check that rules out the most.

Trap: Rebuilding the network before running one diagnostic.

Pillar tie-in: Operational excellence.

Check: A VM can reach an IP but not a hostname. Which step failed?
<details><summary>Answer</summary>Step 1, name resolution.</details>

## Chunk 2: Connectivity Tests
Problem it solves: You need to know whether the configuration allows a path, without logging into either end.

Mental model: Connectivity Tests, in Network Intelligence Center, simulates a packet's path through your VPC, VPN tunnels, and VLAN attachments using your configuration. It names the blocking hop: a firewall rule, a missing route. For some paths it also runs live data plane analysis, sending probes and reporting loss and latency.

Exam signals: "Determine why traffic between two VMs is blocked, without changing anything."

Trap: Opening a broad firewall rule to test. Connectivity Tests answers without changing security.

Pillar tie-in: Operational excellence.

Check: What does Connectivity Tests analyze when it doesn't send live probes?
<details><summary>Answer</summary>The configuration: routes, firewall rules, and other network settings along the simulated path.</details>

## Chunk 3: Flow Logs and firewall rules logging
Problem it solves: You need evidence of what traffic flowed or got blocked.

Mental model: VPC Flow Logs sample connections per subnet and record 5-tuples, bytes, and packets. You set the aggregation interval (5 seconds default, up to 15 minutes) and a secondary sampling rate. Firewall rules logging is per rule; it records connections the rule allowed or denied. Turn it on for the rule you suspect.

Exam signals: "Prove which rule blocked the traffic" points to firewall rules logging. "Analyze traffic volume between subnets" points to Flow Logs.

Trap: Expecting Flow Logs to name the firewall rule. Firewall rules logging does that.

Pillar tie-in: Security and operational excellence.

Check: You need to know which firewall rule denied a connection. What do you enable?
<details><summary>Answer</summary>Firewall rules logging on the suspected deny rule (or on the relevant rules).</details>

## Chunk 4: Health checks and the ranges people forget
Problem it solves: Backends are healthy but the load balancer marks them down.

Mental model: Google health check probes come from `35.191.0.0/16` and `130.211.0.0/22`. The implied deny ingress rule blocks them unless you add an allow rule for those ranges on the serving port.

Exam signals: "All backends show unhealthy", "app works when curled from inside the VPC."

Trap: Debugging the app when the firewall blocks the probes.

Pillar tie-in: Reliability.

Check: Backends answer `curl` from a VM in the same VPC, but the load balancer marks them unhealthy. First check?
<details><summary>Answer</summary>A firewall rule allowing ingress from `35.191.0.0/16` and `130.211.0.0/22` on the health check port.</details>

## Chunk 5: NAT port exhaustion
Problem it solves: Outbound connections from private VMs fail under load.

Mental model: Each VM gets a slice of NAT source ports. A VM opening many connections to one destination IP and port runs out. The NAT metric `dropped_sent_packets_count` shows reason `OUT_OF_RESOURCES`. Fixes: raise minimum ports per VM, turn on dynamic port allocation, or add NAT IPs. `nat_allocation_failed` above 0 means the gateway needs more NAT IPs.

Exam signals: "Intermittent outbound failures at peak", "many connections to one API."

Trap: Scaling up the VM. More CPU doesn't add ports.

Pillar tie-in: Reliability.

Check: Which Cloud NAT metric and reason point to port exhaustion?
<details><summary>Answer</summary>`dropped_sent_packets_count` with reason `OUT_OF_RESOURCES`.</details>

## Chunk 6: Hybrid DNS and MTU
Problem it solves: Two hybrid failures that look like random timeouts.

Mental model:
- DNS forwarding: Cloud DNS sends forwarded queries from `35.199.192.0/19`. On-prem firewalls must allow that range on TCP and UDP 53, and on-prem must route replies back through the VPN or Interconnect.
- MTU: VPC networks default to 1460 bytes, configurable 1300 to 8896. Cloud VPN doesn't fragment after encapsulation, so the peer gateway must pre-fragment. Symptom: small requests work, large transfers hang.

Exam signals: "On-prem forwarding works for some queries, fails from Google Cloud", "SSH works, file copy hangs over VPN."

Trap: Raising timeouts on an MTU problem.

Pillar tie-in: Reliability.

Check: SSH over VPN works but large SCP copies stall. Likely cause?
<details><summary>Answer</summary>MTU. Large packets exceed the path MTU and get dropped; set the peer to pre-fragment or lower the MTU.</details>
