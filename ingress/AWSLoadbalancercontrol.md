***Offical deployment***
- https://github.com/kubernetes-sigs/aws-load-balancer-controller

***Steps***
1. We need service account for k8s workloads communicate with external services.
2. Create policy, use eksctl to create service account
3. Create role and attach the policy.
4. You need OIDC provider. Create one using the document.
   - https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/

***Create an IAM OIDC provider. You can skip this step if you already have one for your cluster.***
```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster roboshop-dev \
    --approve
```
