1. StatefulSet
2. Headless service

- Deployment: For stateless applications. Only shared disks will be created if PV
             and PVC are used.
- StatefulSet: For stateful applicatuons like DB. Every pod replica creates 
               its own disk. Stateful applications in k8s should have cluster IP
               and headless service both. One is for loadbalancing and other one
               is for peer discovery.

***Deployment vs StatefulSet***
1. statefulset is for DB related applications, deployment is for applications.
2. Headless service is mandatory for statefulset, not required for deployment.
3. PV, PVC are mandatory for statefulset.


- What is headless service?
  If a service clusterIP is none, then it is called as headless service. 
  When provisioning stateful applications in k8s we need peer discovery 
  mechanism for data replication. If a pod get create/update/delete request 
  it should update other members as well, for that it uses headless service to know
  the members in cluster, normal service just provides its cluster IP.
  Headless service provides all IP in the cluster.
