***Horizontal scaling vs vertical scaling:***
- HPA: min, desired, max
- avg cpu utilization -> 70% add pods
- What messure the usage? metric server, comes as Add-on for eks.
- Resource requests & limits are mandatory for HPA to messure.
- Without limit we can messure usage.
- HPA always watches the deployment

```
Test HPA
Command: while true; do curl catalogue:8080/health; done
```