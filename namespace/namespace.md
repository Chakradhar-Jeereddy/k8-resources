***NameSpace***
- An isolated project space where we can create our project resources

***PaaS***
- We can create multiple projects in k8s

***Commands***
```
kubectl get namespace
kubectl create namespace roboshop

apiVersion: v1
kind: Namespace
metadata:
 name: chakra
 labels:
   environment: development
   project: roboshop
```

***api versions***
- kubectl api-resources (versions, namespace and cluster scope, alias names)