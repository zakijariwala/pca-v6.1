---
block: B03
title: VPC networking I
pillars: [security, reliability, performance]
exam_guide_refs: ["1.3", "2.1", "3.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/vpc/docs/vpc
  - https://docs.cloud.google.com/vpc/docs/subnets
  - https://docs.cloud.google.com/firewall/docs/firewalls
  - https://docs.cloud.google.com/firewall/docs/firewall-policies-overview
  - https://docs.cloud.google.com/firewall/docs/firewall-policies-rule-eval-order
  - https://docs.cloud.google.com/vpc/docs/routes
  - https://docs.cloud.google.com/nat/docs/ports-and-addresses
  - https://docs.cloud.google.com/vpc/docs/private-google-access
  - https://docs.cloud.google.com/vpc/docs/vpc-peering
  - https://docs.cloud.google.com/vpc/docs/shared-vpc
  - https://docs.cloud.google.com/architecture/best-practices-vpc-design
unverified: []
---
# B03 VPC networking I

## Chunk 1: Global VPC, regional subnets
Problem it solves: You need private IP connectivity across regions without stitching regional networks together.

Mental model: A VPC network is global, with its routes and firewall rules. Subnets are regional. A VM gets its internal IP from a subnet in its region. VMs in different regions on the same VPC talk over internal IPs with no VPN or peering.

On-prem analogy: one routed backbone across every site, with a VLAN per site.

Exam signals: "Multi-region app needs private connectivity", "simplest network design."

Trap: One VPC per region because that's how AWS works. On Google Cloud one VPC spans regions.

Pillar tie-in: Performance and operational excellence.

Check: A VM in `us-east1` and one in `europe-west1` share a VPC. What extra setup do they need to reach each other on internal IPs?
<details><summary>Answer</summary>None beyond firewall rules that allow the traffic. The VPC is global and routes between its subnets exist by default.</details>

## Chunk 2: Auto mode vs custom mode
Problem it solves: IP ranges you didn't choose will collide with on-prem or peered networks later.

Mental model: An auto mode network creates one subnet in every region from predefined ranges inside `10.128.0.0/9`. A custom mode network starts empty and you create subnets with ranges you pick. You can convert auto to custom; not the reverse. Google recommends custom mode for production. Custom mode also supports IPv6 subnet ranges.

Exam signals: "Connect to on-prem later", "avoid overlapping ranges", "production."

Trap: Keeping the default auto mode network for production. Its `10.128.0.0/9` ranges may overlap with on-prem.

Pillar tie-in: Reliability. Overlapping ranges block hybrid connectivity later.

Check: Why does custom mode matter before you build a VPN to on-prem?
<details><summary>Answer</summary>You pick ranges that don't overlap with on-prem. Auto mode's predefined ranges may collide, and overlapping ranges can't route to each other.</details>

## Chunk 3: Firewall rules and policies
Problem it solves: Control what reaches each VM without a firewall appliance in the path.

Mental model: Firewalls are distributed and stateful; they apply at each VM's interface. Every VPC has two implied rules at priority 65535: allow all egress, deny all ingress. Lower numbers win; 0 is highest. At equal priority, deny beats allow.

Layers, evaluated in order:
1. Hierarchical firewall policies at the organization, then folders from top down.
2. Network firewall policies and VPC firewall rules on the network.

A hierarchical rule can allow, deny, or `goto_next` to hand the decision down. Lower levels can't override a higher level's allow or deny.

Target rules by service account or secure tags rather than IP where you can.

Exam signals: "Security team must block a port across every project" points to a hierarchical firewall policy. "Allow only the web tier to reach the app tier" points to rules targeted by service account.

Trap: Adding per-project VPC rules to enforce an org-wide block. A project admin can change those.

Pillar tie-in: Security.

Check: A new custom VPC has no firewall rules you created. Can a VM in it receive SSH from the internet? Send traffic out?
<details><summary>Answer</summary>No inbound SSH: the implied deny ingress rule blocks it. Outbound yes: the implied allow egress rule permits it, if a route and external path exist.</details>

## Chunk 4: Routes
Problem it solves: Packets need a next hop.

Mental model: Each VPC has system-generated subnet routes so subnets reach each other, and a default route to the internet gateway. You can add static routes, and Cloud Router learns dynamic routes over BGP from VPN or Interconnect peers. A route to the internet gateway doesn't give a VM internet access by itself; the VM also needs an external IP or Cloud NAT.

Exam signals: "Send traffic through a firewall appliance" points to a custom static route with the appliance as next hop.

Trap: Deleting the default route to "secure" a network, then wondering why Private Google Access stopped working.

Pillar tie-in: Reliability.

Check: A VM has no external IP and the default route exists. Can it reach the internet?
<details><summary>Answer</summary>No. It needs an external IP or Cloud NAT. The route alone isn't enough.</details>

## Chunk 5: Cloud NAT
Problem it solves: Private VMs need outbound internet for patches and APIs without inbound exposure.

Mental model: Cloud NAT is a managed, regional, software-defined NAT, configured on a Cloud Router. No NAT VM sits in the path. Each NAT IP offers 64,512 TCP and 64,512 UDP source ports. Each VM gets a minimum port allocation: 64 by default with static allocation, 32 with dynamic allocation. Busy VMs that open many connections to the same destination can run out of ports.

Exam signals: "VMs without external IPs need to download updates."

Trap: Giving every VM an external IP for patching. That widens the attack surface.

Pillar tie-in: Security.

Check: How many VMs can one NAT IP support at the static default of 64 ports per VM?
<details><summary>Answer</summary>About 1,008 (64,512 ÷ 64).</details>

## Chunk 6: Private Google Access
Problem it solves: VMs without external IPs still need to call Google APIs such as Cloud Storage.

Mental model: Private Google Access is a per-subnet setting. When on, VMs with only internal IPs in that subnet can reach Google APIs and services. It has no effect on VMs that already have an external IP. Enabling Cloud NAT for a subnet turns on Private Google Access for it too.

Exam signals: "Private VMs must read from Cloud Storage without internet access."

Trap: Adding Cloud NAT when only Google APIs are needed. Private Google Access alone covers it.

Pillar tie-in: Security.

Check: A private VM can't reach `storage.googleapis.com`. What subnet setting do you check first?
<details><summary>Answer</summary>Whether Private Google Access is enabled on the VM's subnet.</details>

## Chunk 7: VPC Peering vs Shared VPC
Problem it solves: Many projects need private connectivity. You choose between central control and independent networks.

Mental model:

| | Shared VPC | VPC Network Peering |
|---|---|---|
| Shape | One network in a host project, used by service projects | Two separate networks joined |
| Admin | Central network team owns subnets, routes, firewalls | Each side runs its own network |
| Org requirement | Same organization | Works across organizations |
| Transitive | N/A (one network) | No. A peers B, B peers C: A can't reach C |

A project can't be both a Shared VPC host and a service project.

Exam signals: "Central network team, app teams manage their own VMs" points to Shared VPC. "Connect to a partner's network in another organization" points to peering.

Trap: Chaining peerings and expecting A to reach C.

Pillar tie-in: Security and operational excellence.

Check: An org wants its network team to own all firewalls while 20 app teams deploy VMs. Shared VPC or peering?
<details><summary>Answer</summary>Shared VPC. The host project holds the network and firewall rules; app teams deploy in service projects.</details>

## Chunk 8: IP planning
Problem it solves: Running out of addresses, or colliding with on-prem, forces a painful rebuild.

Mental model: Plan ranges before you build. Reserve non-overlapping blocks per environment and region. Leave room for GKE Pod and Service secondary ranges, which consume far more addresses than VMs. Record the plan like a DHCP scope sheet for the whole company.

Exam signals: "Expect growth", "GKE clusters", "connect to on-prem 10.0.0.0/8."

Trap: Sizing subnets for today's VM count and forgetting GKE secondary ranges.

Pillar tie-in: Reliability.

Check: What consumes addresses faster than VMs when you plan subnets?
<details><summary>Answer</summary>GKE Pod secondary ranges. Each node reserves a block for Pods.</details>
