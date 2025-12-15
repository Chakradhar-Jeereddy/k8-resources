***Service Account:*** 
- It is a non-human account which pod runs. Service accounts are useful for pod to get access to other resources like ConfigMaps, Secrets and other AWS services like secret manager, parameter store and so on.
- ServiceAccount is the identity used by Pods to access the Kubernetes API, with permissions defined by RBAC.
- Users authenticate via kubeconfig.
- Pods authenticate via ServiceAccounts.
- ServiceAccount = Pod identity

***When a Pod uses a ServiceAccount, Kubernetes automatically:***
- Mounts a JWT token
- Mounts the cluster CA certificate.
- Provides API server endpoint.
```
/var/run/secrets/kubernetes.io/serviceaccount/
```

***Default behavior:***
- Every namespace has a default ServiceAccount.
- If not specified, Pods use default.
- By default, limited permissions.

***Use ServiceAccount in a Pod***
```
spec:
  serviceAccountName: app-sa
```

***Security best practices**
- Create one ServiceAccount per app
- Follow least privilege
- Disable token mount if not needed:
```
automountServiceAccountToken: false
```

***Common Use Cases:***
- Controllers (Ingress, CSI, CNI)
- CI/CD jobs
- Monitoring agents
- Apps calling Kubernetes API                       
