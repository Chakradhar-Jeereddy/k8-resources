### Configuration
- A ConfigMap is an API object(can be immutable) used to store non-confidential data in ***key-value*** pairs.
- Pods can consume ConfigMaps as ***environment variables***, ***command-line arguments***,
  or as configuration files in a ***volume***.
- ConfigMaps are used by Pods to configure containers.

***Purpose:*** Use a ConfigMap for setting configuration data separately from application code.
***Example:*** You developped an application that look into env named DB_HOST. Locally you set 
               the variable to localhost. In clould this env as to refer to service that exposes
               database component to your cluster. In this case, put the variable in configMap 
               and not in Dockerfile, so you don't have to rebuild the image to change the ENV.
               Like cart needs catalogue informatation.
***Differences:*** Unlike most of the k8s objects that have a ***spec***, a configMap has ***data** 
                   and ***bindaryData*** fields. Both accepts key-value pairs as their values
                   Data field accepts UTF-8 strings and binaryData field accepts base64 strings.

***Limitation:*** The data stored in ConfigMap cannot exceed 1 MiB. If you need to store larger then this limit,
                  consider mounting a volume or use a separate database or file service.

***Consumtion:*** Pod spec refers to a ConfigMap and configures containers in the Pod based on data in the ConfigMap.
                  ***Condition:*** Pod/Deployment/statefulset and configMap should be under same namespace.

***Format:***
   - Single key value
   - Keys where values look like fragment of configuration format.
```
data:
 # each key maps to a single value
 player: "3"
 ui_file_name: "user-interface.properties"

 # file like keys
 game.properties: |
  types=aleins, monsters
  lives=5
 interface: |
 color.good=purple
 color.bad=yellow
```

***Different ways to use ConfigMap to configure a container inside Pod:***
- Inside a container command and args
- Environment variables for a container (Updates need pod restart)
- Add a file in read-only volume, for the application to read. (Kubelet updates it at each sync without subPath)
   - Each key in ConfigMap data map becomes the filename under mountPath.
- Write code to run inside pod that uses the k8s API to read a ConfigMap.
```
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
                                     # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # The ConfigMap this value comes from.
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
```
```
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: config
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: game-demo
      # An array of keys from the ConfigMap to create as files
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"
```

***ConfigMap using env variables:***
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  username: k8s-admin
  access_level: "1"
```
*** Pod consumes the content of the ConfigMap as environment variables:***
```
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: myconfigmap
```
***Fetching only one Value***
```
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: CONFIGMAP_USERNAME
      valueFrom:
        configMapKeyRef:
          name: myconfigmap
          key: username
```
***Setting immutable in configmap:***
```
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```
