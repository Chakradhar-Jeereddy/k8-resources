## Volumes
- pod is ephemeral
- node is ephemeral

1. ephemeral
2. Persistent

- When pod is created, it takes some storage from underlying host.
- When pod is deleted, storage is also deleted.

***Ephemeral:***
1. emptyDir
2. hostpath 

- emptyDir: Sidecar pattern. App container logs the transactions, sidecar
            container access it and push logs to external source.
            Both containers share same network and storage.
