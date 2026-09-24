# LAB B04-L01: HA VPN between two VPCs

## Goal
Simulate hybrid connectivity: connect two VPCs with HA VPN (two tunnels, BGP through Cloud Router), ping across, then disable one BGP session and watch traffic keep flowing.

## What reading can't teach
How many pieces an HA VPN needs, what BGP status looks like when healthy and when down, and that one tunnel failing doesn't drop traffic.

## Cost ceiling
Under USD 1.00 for four VPN tunnels, two external IP-less `e2-micro` VMs, and IAP SSH, all under 90 minutes. VPN tunnels bill by the hour. [UNVERIFIED: check Cloud VPN pricing for your region.] Verified: 2026-09-24.

## Time
60 to 90 minutes, including teardown.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Owner or Network Admin plus Compute Instance Admin.
- LAB B03-L01 done.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export ZONE=us-central1-a
export SECRET=$(openssl rand -base64 24)
gcloud config set project "$PROJECT_ID"
gcloud services enable compute.googleapis.com iap.googleapis.com
```

1. Two VPCs with non-overlapping subnets. Call A "cloud" and B "on-prem".
   ```bash
   gcloud compute networks create vpc-a --subnet-mode=custom
   gcloud compute networks subnets create a-sub --network=vpc-a --region="$REGION" --range=10.1.0.0/24
   gcloud compute networks create vpc-b --subnet-mode=custom
   gcloud compute networks subnets create b-sub --network=vpc-b --region="$REGION" --range=10.2.0.0/24
   ```
2. Firewall: IAP SSH, and ICMP from the other side.
   ```bash
   gcloud compute firewall-rules create a-allow --network=vpc-a --direction=INGRESS \
     --rules=tcp:22,icmp --source-ranges=35.235.240.0/20,10.2.0.0/24
   gcloud compute firewall-rules create b-allow --network=vpc-b --direction=INGRESS \
     --rules=tcp:22,icmp --source-ranges=35.235.240.0/20,10.1.0.0/24
   ```
3. HA VPN gateways and Cloud Routers with private ASNs.
   ```bash
   gcloud compute vpn-gateways create gw-a --network=vpc-a --region="$REGION"
   gcloud compute vpn-gateways create gw-b --network=vpc-b --region="$REGION"
   gcloud compute routers create router-a --network=vpc-a --region="$REGION" --asn=65001
   gcloud compute routers create router-b --network=vpc-b --region="$REGION" --asn=65002
   ```
4. Four tunnels: interface 0 and 1 on each side.
   ```bash
   for i in 0 1; do
     gcloud compute vpn-tunnels create tun-a$i --region="$REGION" --vpn-gateway=gw-a --interface=$i \
       --peer-gcp-gateway=gw-b --router=router-a --ike-version=2 --shared-secret="$SECRET"
     gcloud compute vpn-tunnels create tun-b$i --region="$REGION" --vpn-gateway=gw-b --interface=$i \
       --peer-gcp-gateway=gw-a --router=router-b --ike-version=2 --shared-secret="$SECRET"
   done
   ```
5. Router interfaces and BGP peers, one link-local /30 per tunnel pair.
   ```bash
   for i in 0 1; do
     gcloud compute routers add-interface router-a --region="$REGION" --interface-name=if-a$i \
       --vpn-tunnel=tun-a$i --ip-address=169.254.$i.1 --mask-length=30
     gcloud compute routers add-bgp-peer router-a --region="$REGION" --peer-name=peer-a$i \
       --interface=if-a$i --peer-ip-address=169.254.$i.2 --peer-asn=65002
     gcloud compute routers add-interface router-b --region="$REGION" --interface-name=if-b$i \
       --vpn-tunnel=tun-b$i --ip-address=169.254.$i.2 --mask-length=30
     gcloud compute routers add-bgp-peer router-b --region="$REGION" --peer-name=peer-b$i \
       --interface=if-b$i --peer-ip-address=169.254.$i.1 --peer-asn=65001
   done
   ```
6. Check tunnels and BGP. Give it 2 to 3 minutes.
   ```bash
   gcloud compute vpn-tunnels list --format="table(name,status)"
   gcloud compute routers get-status router-a --region="$REGION" \
     --format="table(result.bgpPeerStatus[].name,result.bgpPeerStatus[].status)"
   ```
7. A VM on each side, no external IPs. Ping A → B.
   ```bash
   gcloud compute instances create vm-a --zone="$ZONE" --machine-type=e2-micro --subnet=a-sub --no-address
   gcloud compute instances create vm-b --zone="$ZONE" --machine-type=e2-micro --subnet=b-sub --no-address
   export B_IP=$(gcloud compute instances describe vm-b --zone="$ZONE" --format="value(networkInterfaces[0].networkIP)")
   gcloud compute ssh vm-a --zone="$ZONE" --tunnel-through-iap --command="ping -c 5 $B_IP"
   ```
8. Fail one path: disable BGP on tunnel 0, then ping again.
   ```bash
   gcloud compute routers update-bgp-peer router-a --region="$REGION" --peer-name=peer-a0 --no-enabled
   gcloud compute ssh vm-a --zone="$ZONE" --tunnel-through-iap --command="ping -c 5 $B_IP"
   gcloud compute routers update-bgp-peer router-a --region="$REGION" --peer-name=peer-a0 --enabled
   ```

## Expected output
- Step 6: four tunnels `ESTABLISHED`; both BGP peers `UP`.
- Step 7: 5 packets transmitted, 5 received.
- Step 8: ping still succeeds over tunnel 1.

What to notice: `router-a` learned `10.2.0.0/24` over BGP. You never wrote a static route. Try `gcloud compute routers get-status router-a --region="$REGION"` and look for `bestRoutes`.

## Teardown
Delete in reverse order of creation.

```bash
gcloud compute instances delete vm-a vm-b --zone="$ZONE" --quiet
gcloud compute vpn-tunnels delete tun-a0 tun-a1 tun-b0 tun-b1 --region="$REGION" --quiet
gcloud compute routers delete router-a router-b --region="$REGION" --quiet
gcloud compute vpn-gateways delete gw-a gw-b --region="$REGION" --quiet
gcloud compute firewall-rules delete a-allow b-allow --quiet
gcloud compute networks subnets delete a-sub b-sub --region="$REGION" --quiet
gcloud compute networks delete vpc-a vpc-b --quiet
```

Confirm nothing billable remains:
```bash
gcloud compute vpn-tunnels list
gcloud compute instances list
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
