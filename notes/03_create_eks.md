### EKS by default installs mertics-server
***eksctl create cluster --config-file=eks.yaml***
***eksctl delete cluster --config-file=eks.yaml***
- The yaml file defines the name of the cluster, region,
- managed group: A pool of nodes managed by AWS
- Instance type it can select based on availability
- Desired capacity: 2, 2 workers
- Spot instance true


###eks.yaml
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
    name: roboshop-dev
    region: us-east-1
managedNodeGroups:   # AWS will manage it
  - name: roboshop-dev
    instanceTypes: ["m5.large","t3.small", "t3.medium","c5.large"]
    desiredCapacity: 2 #  by default this value is 3 nodes
    spot: true  # It will create spot instances.
```
