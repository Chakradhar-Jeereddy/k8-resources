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
3. PV, PVC are mandatory for statefulset. Uses volumeclaimtemplate which is like pvc.
4. Pod names are random in deployment, pod names are fixed in statefulset 
5. Pods are created in orderly maner. Statefulest -0/1/2.
6. They are destroyed in orderly manner one by one.
7. It preservs pod names because in case pod crash, new replica will attach to
   same disk/pvc
8. Claims is associated with the pod names. A PVC, PV,disk created for each replica.


- What is headless service?
  If a service clusterIP is none, then it is called as headless service. 
  When provisioning stateful applications in k8s we need peer discovery 
  mechanism for data replication. If a pod get create/update/delete request 
  it should update other members as well, for that it uses headless service to know
  the members in cluster, normal service just provides its cluster IP.
  Headless service provides all IP in the cluster.
