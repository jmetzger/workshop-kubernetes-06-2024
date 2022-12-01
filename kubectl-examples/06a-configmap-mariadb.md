# ConfigMap Example (Mariadb) 

## Schritt 1: configmap 

```
## 01-configmap.yml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: mariadb-configmap 
data:
  # als Wertepaare
  MARIADB_ROOT_PASSWORD: 11abc432
```

```
kubectl apply -f .
kubectl get cm
kubectl get cm -o yaml
```


## Schritt 2: Deployment 
```
cd 
mkdir -p manifests 
cd manifests
mkdir configmapmaria
cd configmapmaria
nano 01-deploy.yml
```

```
#deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
```

```
kubectl apply -f .
```



## Schrit 2: Beispiel als Datei 


```
nano 02-pod.yml
```

```
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-mit-configmap 

spec:
  # Add the ConfigMap as a volume to the Pod
  volumes:
    # `name` here must match the name
    # specified in the volume mount
    - name: example-configmap-volume
      # Populate the volume with config map data
      configMap:
        # `name` here must match the name 
        # specified in the ConfigMap's YAML 
        name: example-configmap

  containers:
    - name: container-configmap
      image: nginx:latest
      # Mount the volume that contains the configuration data 
      # into your container filesystem
      volumeMounts:
        # `name` here must match the name
        # from the volumes section of this pod
        - name: example-configmap-volume
          mountPath: /etc/config


```

```
kubectl apply -f 02-pod.yml 
```

```
#Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-mit-configmap -- ls -la /etc/config
kubectl exec -it pod-mit-configmap --  bash
# ls -la /etc/config 
```

## Schritt 3: Beispiel. ConfigMap als env-variablen 

```
nano 03-pod-mit-env.yml
```

```
# 03-pod-mit-env.yml 
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-env-var 
spec:
  containers:
    - name: env-var-configmap
      image: nginx:latest 
      envFrom:
        - configMapRef:
            name: example-configmap

```

```
kubectl apply -f 03-pod-mit-env.yml
```

```
# und wir schauen uns das an 
#Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-env-var -- env
kubectl exec -it pod-env-var --  bash
# env

```


## Reference: 

 * https://matthewpalmer.net/kubernetes-app-developer/articles/ultimate-configmap-guide-kubernetes.html