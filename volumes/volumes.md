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
