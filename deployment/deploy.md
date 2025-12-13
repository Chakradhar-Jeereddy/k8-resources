### Deployment
1. stop the pods
2. remove old code
3. download new code
4. restart the pods

***Advantage over replicaset***
- Rolling updates
- Rollout history
- Rollback release
- Rolling restart
- Create a pod with version as v2 and remove the old v1.

***Commands:***
- kubectl rollout history deploy ngin-dm
- kubectl rollout history deploy ngin-dm --revision=3
- k rollout undo deploy ngin-dm --to-revision=2
- kubectl set image deployment/nginx-deploy nginx=nginx:bad
- k rollout undo deploy ngin-dm --to-revision=2
- kubectl rollout restart deploy/ngin-dm
- kubectl scale deploy/ngin-dm --replicas=1
