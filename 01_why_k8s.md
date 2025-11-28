***Disadvantages of docker:***
1) No autoscalling
2) No Loadbalancing
3) We can't rely on docker host, it can crash.
4) Hard to manage multiple docker hosts
5) No secrets, poor network managment.
6) No orachestrator to manage the containers in docker hosts
7) All containers should register with overlay network to communicate 
   between containers hosted in multiple hosts.

***Solutions 1:*** 
- Docker swarm -> It is docker's native orchestrator.
- Drawbacks:
   No autscaling in docker swarm.
   LB is not in our control.
   Networking is poor.
   No cloud integration.

***Solution 2:***
- Powerfull and widely used orachestrator in industry.
- Autoscalling, self healing, replicas, rolling updates.
- Efficent network, volumes managment.
- Modular service allows to integrate various tools and features.
- Cloud integration and Cloud LB integration.

***Approach***
1) We build the manifests in personal PC locally and push it to git hub.
2) Download the repo into workstation, pull the changes.
3) Apply the manifests to k8s cluster using the client CLI ***kubectl***

***Tools required to operate k8s***
1) EKSCLI: Update, delete, create k8s cluster in AWS.
2) AWS CLI: Configure authentication and authorisation.
3) KUBECTL: Manage the applications or administrate k8s remotely.
4) Docker: TO build and pull push

***Limatation for Managed k8s***
1) No ssh accress to the control plane.
2) EKSCTL to manage control plane.
3) KUBECTL to manage resource plane/nodes.

***K8s***
- We build the image with docker and run the container using k8s.



   


