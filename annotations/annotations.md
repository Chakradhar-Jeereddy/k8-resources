#### Annotations vs Labels
- Labels: Used to filter resources, there is a limit of key & value length.
          No sepcial charecters are allowed, used to select internal resources.
- Annotations: No limit or more limit char than labels. Sepecial charecters
               are allowed, used to select external resources outside of k8s.
               Cannot be used to select or query objects.
               Allow larger, complex and unstructured data, including char
               not permitted by labels.
               No size restriction.

***Demo***
```
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    jenkins-url: "https://jenkins.joindevops.com/build/job/5"
    imageregistiry: "https://hub.docker.com"
spec:
 containers:
 - name: nginx
   image: nginx
```