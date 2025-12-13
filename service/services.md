### Services
- pop to pod communication throug pod IPs is not useful,
  since pod IPs are ephemeral
- Service provides stable endpoint or IP for pod to pod communication.
- Load balancing
- It acts as DNS to resolve to the backend pods
- Provides single, stable endpoint.

***Example***
- Frontend communicates to catalogue through service of catalogue
- The pods of catalogue gets registered with catalogue service.
- When pods are removed or added, the endpoints gets updated in service.
- All communication to pods happens through service IP and ports.
- Service IP is always stable and never change.
- Pods gets registered with service based on labels and selectors.

***Types of Services***
1) Cluster IP(Default): Exposes the pods within cluster.
2) NodePort: Exposes the pods outside cluster, opens the port on all nodes.
3) Loadbalancer: Forwards the trafic to nodeport and then to cluster IP.

- Cluster IP < NodePort < Loadbalancer

***Drawback of Nodeport***
- We will be exposing the IPs of the nodes of cluster(not secure)
- No load balancing.

***Loadbalancer***
- It routes the trafic to any of the nodes
- Node IPs are hidden from client
- It manages load distribution between nodes.
- Service manages load distribution between pods.
