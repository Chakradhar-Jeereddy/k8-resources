## Volumes
- pod is ephemeral
- node is ephemeral

1. ephemeral
2. Persistent

- When pod is created, it takes some storage from underlying host.
- When pod is deleted, storage is also deleted.

***Ephemeral:***
1. emptyDir
2. hostpath: Ephemeral storage used by pod to access the underlying 
             host storage, this is not secure but in few situations like shipping the logs to extenral storage we are using hostpath with readonly access.
3. Entire EKS cluster is ephemeral, better to keep data outside EKS.

- emptyDir: Sidecar pattern. App container logs the transactions, sidecar
            container access it and push logs to external source.
            Both containers share same network and storage.

 ***Sets:*** - Replicaset
             - Deployment
             - Daemonset -> ensures replica of pod runs in each and 
                            every node. we ship the logs using daemonset.
             - Statefulset

### Mian components the apps need:
1. Networking
2. Storage
3. Programing
4. OS

## Persistent storage
1. EBS - Elastic blockstorage
         Like hard disk, should be close to VM/Computer.
         Data transfer is very fast.
         Can connect to only one computer at a time.
         Database and OS should be in EBS.

2. EFS - Elastic filestorage
         Like NFS, can be anywhere in the network.
         Data transfer is slow complered to EBS.
         Can be used in multiple systems at a time.
         Databases and OS can't be used in EFS.
         You can keep files in EFS.

***Types of provisionings***
1. Static provisioning -> Manually create disks
2. Dynamic provisioning -> Pods can create disk dynamically 
                           whenever required.

Storage administration
=======================
- Create storage and attach to server
- Add or reduce storage to the server
- Remove storage
- Backup and restore data

***K8s admins are not storage admins:*** The wrappers are created to manage 
                                         storage easily.

1. PVC - Claim form/persistent volume claim to mount storage
2. PV - Persistent volume equivalent to disk.
3. StorageClass

***Submit claim-> you get package -> with storage inside***
- Cliam is PVC
- Package is PV
- Storage is underlying disk(EBS/EFS)

Obsolete approach
=================
- Write an email and raise a ticket to storage team asking for some GB like 100GB.
- Get approval from delivery manager and assign them to storage team.
- Someone in the storage team creates the disk.
- You will ask k8 admin to create PV, it is namespace level resource.

PV - It is equivalant object inside k8 that represents physical disk.
     Then roboshop devops engineer creates PVC and attach to pod.

EBS
===
1. Create the volume.
2. Disk should be in same az as the instance
3. Install EBS drivers
4. EC2 instance should have IAM role to access EBS volumes.

***EBS Driver Installation***
```
https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.53"

Objects that gets created:
serviceaccount/ebs-csi-controller-sa created
serviceaccount/ebs-csi-node-sa created
role.rbac.authorization.k8s.io/ebs-csi-leases-role created
clusterrole.rbac.authorization.k8s.io/ebs-csi-node-role created
clusterrole.rbac.authorization.k8s.io/ebs-external-attacher-role created
clusterrole.rbac.authorization.k8s.io/ebs-external-provisioner-role created
clusterrole.rbac.authorization.k8s.io/ebs-external-resizer-role created
clusterrole.rbac.authorization.k8s.io/ebs-external-snapshotter-role created
rolebinding.rbac.authorization.k8s.io/ebs-csi-leases-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-attacher-binding created
clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-node-getter-binding created
clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-provisioner-binding created
clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-resizer-binding created
clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-snapshotter-binding created
deployment.apps/ebs-csi-controller created
poddisruptionbudget.policy/ebs-csi-controller created
daemonset.apps/ebs-csi-node created
csidriver.storage.k8s.io/ebs.csi.aws.com created
```
***EBS CSI driver has two main components:***
1. Controller (runs in control plane / worker node)
2. Node Plugin (runs on every worker node as a DaemonSet)
  ***Main responsibilities:***
      - Create EBS volumes
      - Delete EBS volumes
      - Attach EBS volumes to EC2 instances
      - Detach EBS volumes
      - Create / delete snapshots
      - Handle volume expansion
  ***Triggered by:***
    - PersistentVolumeClaim (PVC) creation
    - StorageClass parameters
  ***Controller talks to:***
    - AWS EC2 API
    - Kubernetes API Server
    - When PVC created → Controller creates EBS volume → attaches it to the node
***EBS CSI Node Plugin – “Mount & unmount volumes”***
    - Runs as a DaemonSet (one pod per node).
   ***Main responsibilities:***
       - Format the EBS volume
       - Mount volume into the node
       - Bind-mount into the Pod
       - Unmount volume when Pod is deleted
       - Node-level volume expansion
  ***Node plugin talks to:***
     - kubelet
     - OS mount utilities
     - Volume attached → Node plugin mounts /dev/xvdf → Pod sees /data
     
***Simple analogy (easy to remember)***
  - Component	Role
  - Controller	📦 Create / Attach / Delete EBS
  - Node Plugin	🔧 Mount / Unmount / Format
    
***Static Provisioning:***
- Frist create EBS volume in an AZ, add EBS permissions to IAM role of EC2 instance.
- EC2-> Security->IAM role->Add permissions-> attach police->AmazonEBSCSIDriverPolicy
- Create PV using the volume ID
- Create PVC
- Create pod to mount the PVC
- Use node selectors to make sure pod is scheduled on node the volume is attached.

***Dynamic Provisioning:***
- storageClass -> Used for dynamic provisioning to create the PV & Disk automatically.
- Example: https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/examples/kubernetes/dynamic-provisioning/manifests/storageclass.yaml

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
```

EFS
===
- EFS is elastic and size can go upto PB, it has max filesize 47.9 TB.
- It automatically expands its size as needed, you for what used.
- We can select fs type in EBS, but EFS has fixed type called NFS.
- EBS not requied SG, for EFS we should attach security group(its in network).
- NFS port is 2049
- Allow 2049 in EFS SG from EC2 SG

EFS Static provisioning
=======================
1. Create disk.
2. Install the EFS drivers.
```
kubectl kustomize \
    "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-2.1" > public-ecr-driver.yaml
k apply -f public-ecr-driver.yaml
```
3. EC2 nodes should have IAM roles to access and mount EFS volumes.
4. EFS-> Create EFS->Name: roboshop-> EKS VPC-> Create.
5. IAM Roles -> add permissions->attach policy->AmazonEFSCSIDriverPolicy->add.
