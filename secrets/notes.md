### Secrets
- Using a Secret means that you don't need to include confidential data in your application code.
- You can make secret visible only to one container in Pod, mount using volumeMounts only in one.
- Data is not stored in encrypted format in etcd. Need to take additional considerations.
***different ways to mount/use a Secret in a Pod:***
```
As env(all keys)
envFrom:
- secretRef:
    name: app-secret
```
```
As env(specific key)
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: password
```
```
As files using a volume (MOST COMMON)
Each key becomes a file.

volumes:
- name: secret-vol
  secret:
    secretName: app-secret

volumeMounts:
- name: secret-vol
  mountPath: /etc/secret

```
```
Mount specific keys only
secret:
  secretName: app-secret
  items:
  - key: username
    path: db-user
```
```
Control file permissions

secret:
  secretName: app-secret
  defaultMode: 0400
```
```
Optional Secret (Pod starts even if missing)
secretKeyRef:
  name: app-secret
  optional: true
```
```
Mount a single Secret file using subPath
volumeMounts:
- name: secret-vol
  mountPath: /etc/secret/password
  subPath: password
```
```
Update behaviour
| Method       | Auto update |
| ------------ | ----------- |
| Env vars     | ❌ No        |
| Volume mount | ✅ Yes       |
| subPath      | ❌ No        |
```
```
CKA one-liners
Secrets are base64 encoded, not encrypted
Secrets are namespace-scoped
Prefer volume mounts over env vars for rotation
Don’t store secrets in Dockerfiles
```
***More security***
  1. Mounts secrets from:
      - AWS Secrets Manager
      - Azure Key Vault
      - HashiCorp Vaul


- Below creates a hidden file and stores password in that file
```
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      command:
        - ls
        - "-l"
        - "/etc/secret-volume"
      volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"
```
