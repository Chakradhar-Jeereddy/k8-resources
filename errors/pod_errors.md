- ImagePullBackoff -> Image address wrong
- crashLoopBackoff -> Unable to start container due to configuration error

***Test***
```
apiVersion: v1
kind: Pod
metadata:
 name: errortest
spec:
 containers:
<!--  - name: nginx
   image: nginxerrrr -->
 - name: almalinux
   image: almalinux
```