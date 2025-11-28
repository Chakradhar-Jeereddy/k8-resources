### EKS by default installs mertics-server
***eksctl create cluster --config-file=eks.yaml***
- The yaml file defines the name of the clyster, region,
- managed group: A pool of nodes managed by AWS
- Instance type it can select based on availability
- Desired capacity: 2, 2 workers
- Spot instance true

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