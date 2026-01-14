- ***Static provisioning***
- ***Dynamic provisioning***

Static
======
1. Install drivers and create the disk.
2. Make sure nodes have IAM access to EBS/EFS.
3. If EFS, SG should allow port no 2049 from worker nodes.
4. If EBS volume should be in the same AZ as server.
5. We need to create PV, PVC and mount to the pod.

Dynamic
=======
1. Install drivers
2. Make sure nodes have IAM access to EBS/EFS.
3. IF EFS SG should allow port no 2049 from worker nodes.
4. We need to create SC.
5. We need to create PVC, mention SC name.
6. Volume and PV will be automatically created.

- **provisioningMode: efs-ap** is a setting used in Kubernetes StorageClass when you are working with Amazon EFS using the EFS CSI driver.
- What it means (in simple terms)
   - It tells Kubernetes to dynamically create an EFS Access Point for each PersistentVolumeClaim (PVC).

Breakdown
==
- EFS = Amazon Elastic File System (shared NFS storage)
- AP = Access Point
- efs-ap = Use EFS Access Points for provisioning
- So instead of manually creating directories in EFS, Kubernetes:
- Creates an EFS Access Point
- Creates a dedicated directory in EFS
- Applies POSIX permissions automatically
- Isolates storage per PVC

SC definaion:
==
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-12345678
  directoryPerms: "700"
```

What happens when a PVC is created?
==
- You create a PVC
- Kubernetes calls the EFS CSI driver
- The driver:
- Creates an EFS Access Point
- Creates a directory like:
- /pvc-<uuid>
- Sets ownership & permissions
- A PersistentVolume is automatically created and bound

Why this is useful
==
✅ No manual EFS directory management
✅ Better security & isolation per workload
✅ Works perfectly with dynamic provisioning
✅ Recommended by AWS for EKS + EFS

Access point
==
- EFS Access Point like a private door into a shared storage room
- EFS = One big shared hard drive (like a common store room)
- Access Point = A specific door to that store room
- Each door opens into its own folder
- Has fixed permissions
- Has fixed owner (UID/GID)
- Everyone uses the same store room, but each person enters through their own door and sees only their space.

Without Access Point
==
- Pods mount the root of EFS:
```
/
├── app1
├── app2
├── app3
```

***Problems:**

- Permissions conflict
- Apps can accidentally read/write others’ data
- Manual setup needed

With Access Point
==
- Each pod gets its own directory:
```
/
├── pvc-1111  ← Access Point 1
├── pvc-2222  ← Access Point 2
```
- Each Access Point locks pod to its directory
- Enforces permissions automatically

Kubernetes automatically:
==
- Creates an Access Point
- Creates a directory
- Sets permissions
- Mounts only that directory to the pod
👉 No manual work
👉 No permission headaches
👉 Safer multi-pod usage

Dynamic volume resuze for EFS
==
- EFS does NOT need dynamic volume resize. It grows automatically.
**Why?**
- EFS is elastic (no fixed size)
- You don’t pre-allocate storage
- You pay only for what you use
- There is no “disk full” scenario like EBS

**What happens in Kubernetes**
- For EBS (block storage):
- You must resize the PVC
- Kubernetes expands the volume
- For EFS (file storage):
- PVC size is just informational
- You can write beyond the PVC size
- EFS keeps growing automatically
  *Example:*
  ```
  resources:
  requests:
    storage: 5Gi
  ```
- Even if you write 100Gi, EFS is fine ✅

Why PVC still asks for size?
==
- Kubernetes requires a size field, but:
- EFS CSI driver ignores it
- No resize operation happens
- **Key takeaway (one line)**
  - **Dynamic volume resize doesn’t apply to EFS — it auto-scales by design.**

Quick comparison
Storage	Resize needed?	How
EBS	✅ Yes	Edit PVC
EFS	❌ No	Auto grows

***In Azure Files, you DO need to resize — it does NOT auto-grow like EFS.***
- Compare first (EFS vs Azure Files)
- Feature	AWS EFS	Azure Files
- Storage type	File	File
- Auto-grows	✅ Yes	❌ No
- PVC resize needed	❌ Not applicable	✅ Yes
- Kubernetes support	Size ignored	Size enforced
  
Azure Files behavior (simple terms)
==
- Think of Azure Files like a fixed-size network drive.
- You create a file share with a quota
- Once quota is full → writes fail
- To get more space → increase quota
- So dynamic resize is required.

**Kubernetes + Azure Files**
 **Azure Files supports dynamic PVC resize, but only if enabled**

1️⃣ StorageClass must allow expansion
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile-sc
provisioner: file.csi.azure.com
allowVolumeExpansion: true
```

Pod restart?
==
- ❌ Usually not required
- Azure Files is filesystem-based
  - Resize is online

One-line takeaway
==
**Azure Files requires PVC resize because it has a fixed quota — unlike EFS which auto-scales.**

Why shrinking doesn’t exist in EFS
==
- EFS is elastic file storage, not a fixed-size volume.
- There is no allocated size
- Storage = actual data stored
- You pay per GB used
- So there’s nothing to “resize down”.


What does work instead
==
**If you want to reduce EFS usage (and cost), you must:**
- 1️⃣ Delete data
- Remove files/directories you don’t need
- Storage usage drops automatically
- 2️⃣ Delete unused Access Points
- Access points themselves are free
- But the directories they point to may contain data
- 3️⃣ Use Lifecycle Policies
- Move cold data to EFS Infrequent Access (IA):
- Cheaper storage
- Still accessible

One-line takeaway
==
**EFS doesn’t shrink — you reduce usage by deleting data, not resizing volumes.**

✅ For EBS / Azure Disk / Azure Files
==
- Resize actually happens
- Underlying storage grows
- Sometimes pod restart needed (block storage)

❌ For EFS
==
- PVC size is ignored
- EFS grows automatically
- Expansion flag has no real effect
**So adding it for EFS is harmless but meaningless.**

**Example storage class**
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: ebs.csi.aws.com
allowVolumeExpansion: true
```

Does EBS support dynamic shrinking?
==
Clear answer

EBS supports dynamic expansion ✅

EBS does NOT support shrinking ❌

This is true for Kubernetes, AWS, and EBS itself


Why shrinking is not supported
==
**Shrinking a block device is risky:**
- High chance of data corruption
- Filesystems must be resized offline
- AWS does not allow decreasing EBS volume size
- So AWS + Kubernetes block shrinking entirely.


How to “shrink” EBS (manual workaround)
==
**Only option is data migration:**
- Create a new smaller PVC
- Copy data:
- rsync -av /old /new
- Update pod to use new PVC
- Delete old PVC

**⚠️ Downtime required in most cases**


Comparison table
==
- ***Storage	Expand	Shrink**
| Storage Type   | Expand | Shrink |
|----------------|--------|-------|
| EBS            | ✅      | ❌     |
| EFS            | Auto    | N/A    |
| Azure Disk     | ✅      | ❌     |
| Azure Files    | ✅      | ❌     |
Big picture
==
- **Filesystem	Online grow	Online shrink**
| Filesystem | Online Grow | Shrinking |
|------------|------------|-----------|
| ext4       | ✅ Yes     | ❌ No (offline only) |
| XFS        | ✅ Yes     | ❌ Never supported   |


Real-world recommendation
==
| Use case | Best filesystem |
|--------|----------------|
| General workloads | ext4 |
| Large / high-throughput workloads | XFS |
| Might need shrink | ext4 (manual only, offline) |
| Kubernetes workloads | Either (expand only) |
