***Service Account:*** 
- It is a non-human account which pod runs. Service accounts are useful for pod to get access to other resources like ConfigMaps, Secrets and other AWS services like secret manager, parameter store and so on.
- ServiceAccount is the identity used by Pods to access the Kubernetes API, with permissions defined by RBAC.
- Users authenticate via kubeconfig.
- Pods authenticate via ServiceAccounts.
- ServiceAccount = Pod identity

Declarative
==
```
apiVersion: v1            # Application version of the resource.
kind: ServiceAccount      # Declares the resource a service account.
metadata:                 # Metadata of the resource
  name: shadow-sa         # Name of the service account
  namespace: default      # Declares the namespace the service account belongs to.
---
apiVersion: v1            # Application version of the resource.
kind: Pod                 # Declares the resource as a pod.
metadata:                 # Metadata of the resource.
  name: mypod             # Name of the pod resource
  namespace: default
spec:                     # Defines the desired state of the service account
  # automountServiceAccountToken: false  # Disabling the access to k8s API server
 restartPolicy: Always                # Restart policy is always at pod level and not per container (expect initContainer)
 containers:              # List of containers
 - name: nginx
   image: nginx:latest
   imagePullPolicy: Always      # Always pull image from report repository
   resources:
     limits:
       cpu: "10m"
       memory: "100Mi"
     requests:
       cpu: "10m"
       memory: "100Mi"
```

***When a Pod uses a ServiceAccount, Kubernetes automatically:***
- Mounts a JWT token
- Mounts the cluster CA.crt certificate.
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

Jenkins using Service account
==
- Create a jenkins-sa account.
- Create a role with permissions (verbs)
- Create rolebinding to bind the role to service account.
- Create a pod manifest with serviceAccountName: jenkins-sa in specs section.
- Below is the flow
- Jenkins -> K8s API server
- Auth using JWT
- Create pods, run the pipeline
- Delete pods after the pipeline exectution.

Json web token (JWT)
==
- JWT token is a singned identity card that kubernetes gives to POD.
- To know who they are and what they can do.
- No kubeconfig,no user/password store.
- Short lived, token is rotated in newer version of k8s.
- It can't be tampared. It has an expiry date/
- The token include, namespace, expiry, permissions and SA.

automountServiceAccountToken
==
- If automountServiceAccountToken: false.
- K8s will not inject JWT token into pod.
- Pod/app cannot communicate with kube API.
- The application logic, network calls can run.
- Use cases
   - Frontend app
   - Backend rest API
   - database jobs.

Can pod still get access somehow?
===
- Yes, only if you provide credentials manually:
- Mount kubeconfig as secret (not recommended)
```
volumeMounts:
- name: kubeconfig
  mountPath: /root/.kube/config
```
