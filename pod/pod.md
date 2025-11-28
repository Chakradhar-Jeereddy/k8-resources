***Pod:*** Smallest deployable unit in kubernetes
           We can run one or more containers in pod.
           We can't directly create containers in k8s.
           Containers inside pod share same network and volume
           All containers inside pod will have same IP.

***Side car pod:*** Nginx container logs in ephemeral volume
                    logger will push logs to external storage
                    Both will run in same pod.
