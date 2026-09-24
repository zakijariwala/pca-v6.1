---
block: B05
title: Compute Engine
pillars: [reliability, cost, performance]
exam_guide_refs: ["1.3", "2.3"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/compute/docs/machine-resource
  - https://docs.cloud.google.com/compute/docs/instances/spot
  - https://docs.cloud.google.com/compute/docs/instance-groups
  - https://docs.cloud.google.com/compute/docs/instance-groups/autohealing-instances-in-migs
  - https://docs.cloud.google.com/compute/docs/nodes/sole-tenant-nodes
  - https://docs.cloud.google.com/compute/docs/nodes/bringing-your-own-licenses
  - https://docs.cloud.google.com/compute/docs/disks/snapshots
  - https://docs.cloud.google.com/compute/docs/disks/instant-snapshots
  - https://docs.cloud.google.com/compute/docs/oslogin
unverified: []
---
# B05 Compute Engine

## Chunk 1: Machine families
Problem it solves: The wrong machine shape wastes money or starves the workload.

Mental model: Families by workload shape:

| Family | For |
|---|---|
| General-purpose | Most workloads; best price-performance |
| Compute-optimized | High performance per core, HPC |
| Memory-optimized | Large in-memory databases; up to multi-TB memory |
| Accelerator-optimized | GPU workloads such as ML training and HPC |
| Storage-optimized | High storage density, low core use |
| Network-optimized | High network throughput |

Custom machine types let you pick vCPU and memory within a series when predefined shapes don't fit.

Exam signals: "SAP HANA" points to memory-optimized. "Model training on GPUs" points to accelerator-optimized. "Balanced web tier" points to general-purpose.

Trap: Picking compute-optimized for a database that needs RAM.

Pillar tie-in: Cost and performance.

Check: A workload needs 6 vCPUs and 40 GB RAM, and no predefined type fits. What do you use?
<details><summary>Answer</summary>A custom machine type in a series that supports it.</details>

## Chunk 2: Instance templates and managed instance groups
Problem it solves: Hand-built VMs are pets. You need identical, replaceable cattle.

Mental model: An instance template is the VM blueprint: machine type, image, disks, network, metadata. A managed instance group (MIG) runs N copies from a template. It autoscales on load, autoheals with a health check, and rolls out new templates. A regional MIG spreads VMs across zones in a region, within one VM per zone of each other.

Autohealing uses an application health check. The initial delay (0 to 3600 seconds, default 0) gives new VMs time to boot before failed checks count.

Exam signals: "Survive a zone failure" points to a regional MIG. "Replace unhealthy VMs automatically" points to autohealing.

Trap: Initial delay left at 0 for an app that takes 3 minutes to start. The MIG recreates VMs in a loop.

Pillar tie-in: Reliability.

Check: A MIG keeps recreating VMs that later turn out healthy. What setting do you check?
<details><summary>Answer</summary>The autohealing initial delay. It's shorter than the app's startup time, so the MIG acts on failed checks during boot.</details>

## Chunk 3: Spot VMs
Problem it solves: Batch and fault-tolerant work shouldn't pay on-demand prices.

Mental model: Spot VMs run on spare capacity at 60 to 91% off on-demand. Google can reclaim them at any time, with up to 30 seconds of notice (best effort). They have no maximum runtime unless you set one. Legacy preemptible VMs stop after 24 hours; Spot VMs replace them.

Exam signals: "Batch processing", "can tolerate interruption", "minimize cost."

Trap: Spot VMs for a stateful database or anything with an SLA.

Pillar tie-in: Cost.

Check: A nightly render job can restart from checkpoints. What VM option cuts cost most?
<details><summary>Answer</summary>Spot VMs, with checkpointing to handle preemption.</details>

## Chunk 4: Sole-tenant nodes
Problem it solves: Licensing or compliance demands dedicated physical hardware.

Mental model: A sole-tenant node is a physical server that hosts VMs for your project only. Use it for bring-your-own-license software licensed per physical core or processor, or when a regulator requires physical isolation. You're responsible for picking a tenancy model that fits your license terms.

Exam signals: "Per-core Windows Server or SQL Server licenses", "no other customer on the host."

Trap: Sole-tenancy as a security upgrade for ordinary workloads. It costs more and seldom fits the requirement.

Pillar tie-in: Cost and security.

Check: A company brings per-physical-core licenses from on-prem. What host option keeps them compliant?
<details><summary>Answer</summary>Sole-tenant nodes.</details>

## Chunk 5: Disks and snapshots
Problem it solves: Data must survive VM loss, zone loss, and human error.

Mental model: Standard snapshots are incremental, geo-redundant backups stored as global resources; you can restore them into a disk in any region. Snapshot schedules create them hourly, daily, or weekly. Instant snapshots are created in seconds but stay in the disk's zone or region; schedules can't create them. Instant snapshots suit fast rollback; standard snapshots suit DR.

Exam signals: "Restore in another region" points to standard snapshots. "Roll back quickly before a risky change" points to instant snapshots.

Trap: Relying on instant snapshots for regional DR. They live in the same location as the disk.

Pillar tie-in: Reliability.

Check: You need a daily backup you can restore in a different region. Standard or instant snapshot?
<details><summary>Answer</summary>Standard snapshot on a snapshot schedule. It's global and geo-redundant.</details>

## Chunk 6: Startup scripts and OS Login
Problem it solves: Configuring VMs at boot, and controlling who can SSH in.

Mental model: A startup script in metadata runs at every boot. Keep it idempotent. OS Login ties SSH access to IAM: grant a role and the user can SSH; remove it and they can't. With OS Login on, the VM ignores SSH keys stored in metadata.

Exam signals: "Centrally manage SSH access", "revoke access when someone leaves."

Trap: Managing SSH keys in project metadata for a large team.

Pillar tie-in: Security and operational excellence.

Check: An engineer leaves. With OS Login on, how do you remove their SSH access to every VM?
<details><summary>Answer</summary>Remove their IAM role (or group membership). OS Login checks IAM, so access ends everywhere at once.</details>

## Chunk 7: VMs vs containers vs serverless
Problem it solves: Knowing when a VM is the right answer.

Mental model: Choose a VM when you need OS control, a specific kernel or license, GPUs with custom drivers, lift-and-shift of software you can't containerize, or long-running stateful processes. Otherwise, containers or serverless cut ops burden. B06 builds the full decision tree.

Exam signals: "Legacy app, can't be modified", "needs a specific OS version."

Trap: Forcing an unmodifiable legacy app into Cloud Run.

Pillar tie-in: Operational excellence.

Check: Name two signals that point to Compute Engine over Cloud Run.
<details><summary>Answer</summary>Any two of: needs OS or kernel control, licensed software tied to hosts, can't be containerized, long-running stateful process, special drivers.</details>
