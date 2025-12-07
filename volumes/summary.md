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
