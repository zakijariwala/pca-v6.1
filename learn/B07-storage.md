---
block: B07
title: Storage
pillars: [cost, reliability, security]
exam_guide_refs: ["1.3", "2.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/storage/docs/storage-classes
  - https://docs.cloud.google.com/storage/docs/locations
  - https://docs.cloud.google.com/storage/docs/lifecycle
  - https://docs.cloud.google.com/storage/docs/autoclass
  - https://docs.cloud.google.com/storage/docs/object-versioning
  - https://docs.cloud.google.com/storage/docs/soft-delete
  - https://docs.cloud.google.com/storage/docs/bucket-lock
  - https://docs.cloud.google.com/storage/docs/object-lock
  - https://docs.cloud.google.com/storage/docs/access-control/signed-urls
  - https://docs.cloud.google.com/compute/docs/disks
  - https://docs.cloud.google.com/compute/docs/disks/hyperdisks
  - https://docs.cloud.google.com/compute/docs/disks/local-ssd
  - https://docs.cloud.google.com/filestore/docs/service-tiers
  - https://docs.cloud.google.com/netapp/volumes/docs/discover/overview
unverified: []
---
# B07 Storage

## Chunk 1: Cloud Storage classes
Problem it solves: Storing petabytes at one price wastes money on data nobody reads.

Mental model: All classes share the same API and millisecond access. They differ in storage price, retrieval fees, and minimum storage duration.

| Class | Minimum storage duration | Fits data read |
|---|---|---|
| Standard | None | Often |
| Nearline | 30 days | About once a month |
| Coldline | 90 days | About once a quarter |
| Archive | 365 days | Less than once a year |

Delete or move an object before its minimum duration and you still pay for the full duration. Colder classes charge retrieval fees. All classes carry 99.999999999% (eleven nines) annual durability.

Exam signals: "Retain for 7 years, read only for audits" points to Archive. "Backups restored monthly" points to Nearline.

Trap: Archive because "it's cheapest," for data read every week. Retrieval fees erase the savings.

Pillar tie-in: Cost.

Check: Archive data is slow to retrieve, like tape. True or false?
<details><summary>Answer</summary>False. All Cloud Storage classes give millisecond access. Archive costs more to read and has a 365-day minimum, but it isn't slow.</details>

## Chunk 2: Location types
Problem it solves: Choose how many regions hold your data, trading cost against availability and DR.

Mental model:

| Type | Where | Fits |
|---|---|---|
| Region | One region | Data next to compute, lowest cost |
| Dual-region | A specific pair of regions | Regional DR with control over both locations |
| Multi-region | A large area such as US or EU | Content served across a continent |

Dual-regions and multi-regions are geo-redundant. Dual-regions offer turbo replication, which replicates new objects within 15 minutes. Standard class availability runs 99.99% in regions and higher than 99.99% in dual-regions and multi-regions.

Exam signals: "RPO of 15 minutes for object data across regions" points to dual-region with turbo replication. "Must stay in Germany" rules out an EU multi-region if it spans other countries.

Trap: A multi-region for data processed by compute in one region. You pay more and add cross-region latency.

Pillar tie-in: Reliability and cost.

Check: A bucket must survive a region outage with a 15-minute RPO. Which location type and option?
<details><summary>Answer</summary>Dual-region with turbo replication.</details>

## Chunk 3: Lifecycle rules and Autoclass
Problem it solves: Moving aging data to colder classes by hand never happens.

Mental model: Lifecycle rules apply actions (`Delete`, `SetStorageClass`, `AbortIncompleteMultipartUpload`) when conditions match: age, creation date, number of newer versions, live or noncurrent. Rules run asynchronously; config changes can take up to 24 hours to take effect.

Autoclass manages classes per object based on access. Objects not read for 30 days move to Nearline, 90 days to Coldline, 365 days to Archive (if you set Archive as the terminal class; default terminal class is Nearline). A read moves the object back to Standard.

Exam signals: "Access pattern unknown" points to Autoclass. "Delete logs after 90 days" points to a lifecycle rule.

Trap: A lifecycle rule that moves data to Coldline, then an app that reads it daily.

Pillar tie-in: Cost and operational excellence.

Check: When access patterns are unpredictable, which feature picks classes for you?
<details><summary>Answer</summary>Autoclass.</details>

## Chunk 4: Protecting objects: versioning, soft delete, retention
Problem it solves: Accidental deletes, ransomware, and regulators who say "keep it, unchanged, for 7 years."

Mental model:

| Feature | Protects against | Note |
|---|---|---|
| Object versioning | Overwrite and delete | Keeps noncurrent versions; pair with lifecycle rules to cap cost |
| Soft delete | Accidental delete | On by default, 7-day retention, configurable 7 to 90 days |
| Retention policy + Bucket Lock | Early delete or change, bucket-wide | Once locked, the policy can't be reduced or removed |
| Object Retention Lock | Early delete, per object | Retention per object; a locked retention can't be shortened |

Bucket Lock helps meet record-keeping rules such as those from FINRA, SEC, and CFTC.

Exam signals: "WORM", "regulator requires immutable records" point to a locked retention policy.

Trap: Treating versioning as compliance. Someone with the right role can still delete versions. A locked retention policy blocks that.

Pillar tie-in: Security and reliability.

Check: After you lock a bucket's retention policy, can you shorten it?
<details><summary>Answer</summary>No. A locked retention policy can't be reduced or removed.</details>

## Chunk 5: Signed URLs
Problem it solves: Give someone without a Google account time-limited access to one object.

Mental model: A signed URL embeds a signature and expiry. Anyone with the URL can read (or upload, if signed for it) until it expires. V4 signed URLs last at most 7 days (604,800 seconds).

Exam signals: "Customers download their invoice PDF without logging in", "partner uploads a file once."

Trap: Making the bucket public so one partner can fetch a file.

Pillar tie-in: Security.

Check: What's the longest a V4 signed URL can stay valid?
<details><summary>Answer</summary>7 days.</details>

## Chunk 6: Block storage: Persistent Disk, Hyperdisk, Local SSD
Problem it solves: VMs need disks, and the right type depends on durability and performance.

Mental model:

| Type | Durable | Notes |
|---|---|---|
| Persistent Disk | Yes | Network block storage; zonal or regional |
| Hyperdisk | Yes | Set IOPS and throughput per volume; Hyperdisk Balanced High Availability replicates across two zones |
| Local SSD | No | Attached to the host; fastest; data may be lost when the VM stops |

Exam signals: "Scratch space for temporary data, highest IOPS" points to Local SSD. "Tune IOPS independent of size" points to Hyperdisk.

Trap: A database on Local SSD with no replication.

Pillar tie-in: Performance and reliability.

Check: Which block storage type can lose data when the VM stops?
<details><summary>Answer</summary>Local SSD.</details>

## Chunk 7: File storage: Filestore and NetApp Volumes
Problem it solves: Apps that expect a shared POSIX file system can't use object storage without rework.

Mental model: Filestore is managed NFS. Tiers include Zonal (performance, localized), Regional (survives a zone outage), and Enterprise multishares for GKE; Basic HDD and Basic SSD remain as legacy tiers. Google Cloud NetApp Volumes is managed NetApp storage with NFS, SMB, iSCSI, and more, for lifting enterprise NAS workloads, including Windows file shares.

Exam signals: "Lift-and-shift app needs NFS" points to Filestore. "Windows SMB shares" or "existing NetApp" points to NetApp Volumes.

Trap: Rewriting a legacy app to use Cloud Storage when an NFS mount keeps it running as-is.

Pillar tie-in: Operational excellence.

Check: A migrating Windows app needs SMB file shares. Filestore or NetApp Volumes?
<details><summary>Answer</summary>NetApp Volumes. It supports SMB; Filestore is NFS.</details>
