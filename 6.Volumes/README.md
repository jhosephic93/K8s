# K8s Volumes:

- Volume Types
    - EmptyDir
        - Emptydir es un volume al vuelo no tiene lugar, se guarda en memoria, es efimero.
    - NFS
        - No se hizo el ejempo porque se necesita un Server NFS.
    - HostPath
        - Se crea en una ruta especifica del nodo de Kubernetes Cluster.
    - ConfigMap
      - Almacena valores de configuracion (key-value pairs)
      - Valores consumidos en pods como: (1. Environment variables 2.Files)
      - Ayuda a separar el codigo de la configuracion.
    - Secret
      - Para almacenar y administar data sensble, (passwords, tokens, keys)
      - Referenciados como archivos en un volume, montado de un secret.
      - Base64 encoded.
      - Types: generic, Docker registry, TLS

**************************************

## HostPath

    - Persistent Volume
    - Persistent Volume Claim:
    - Pod que use el Claim:

## Persistent Volume:

1. Crear archivo pv.yaml con contenido:

```console
$ nano pv.yaml
```

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: my-pv
  labels:
    type: local
spec:
  # Match con el pvc
  storageClassName: manual
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    # Se ejecuta en nodo y debe tener permisos
    path: "/tmp/datapv"
```

2. Crear y verificar:

```console
$ kubectl apply -f pv.yaml 
$ kubectl get persistentvolume
$ kubectl get pv
$ kubectl describe persistentvolume my-pv
```

### Persistent Volume Claim:

3. Crear archivo pvc.yaml con contenido:

```console
$ nano pvc.yaml
```

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  # Match con el pv
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Mi
```

4. Crear y verificar:

```console
$ kubectl apply -f pv.yaml 
$ kubectl get persistentvolumeclaim
$ kubectl get pvc
$ kubectl describe persistentvolumeclaim/myclaim
```

### Pod que use el Claim:

5. Crear un pod_pv_pvc.yaml con contenido:

```console
$ pod_pv_pvc.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: alpine
spec:
  volumes:
    - name: mi-volume
      persistentVolumeClaim:
        claimName: myclaim
  containers:
    - image: alpine
      name: alpine
      command:
        - sleep
        - "3600"
      volumeMounts:
      - mountPath: /mnt/mydata
        name: mi-volume
```

6. Crear y verificar:

```console
$ kubectl apply -f pod_pv_pvc.yaml
$ kubectl get pods
$ kubectl exec -ti alpine sh
# mkdir /mnt/mydata/hello
# echo "xD" > /mnt/mydata/hello/hi.txt
# exit
```

7. Eliminar pod y probar persistencia:

```console
$ kubectl delete -f pod_pv_pvc.yaml
$ kubectl apply -f pod_pv_pvc.yaml
$ kubectl exec -ti alpine sh
# cat /mnt/mydata/hello/hi.txt
# exit
```

8. Verificar el hostPath en el K8s cluster:

```console
$ minikube ssh
$ ls -la /tmp/datapv/hello/
$ cat /tmp/datapv/hello/hi.txt
$ exit
```

**************************************

## Volumen emptyDir:

1. Crear archivo pod_emptydir.yaml con contenido:

```console
$ nano pod_emptydir.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-empty
spec:
  volumes:
    - name: scratch-volume
      emptyDir: {}
  containers:
    - image: alpine
      name: alpine
      command:
        - sleep
        - "3600"
      volumeMounts:
      - mountPath: /scratch
        name: scratch-volume
```

2. Crear y verificar:

```console
$ kubectl apply -f pod_emptydir.yaml
$ kubectl get pod
$ kubectl exec -ti alpine-empty sh
# echo "xD" > /scratch/hello.txt
# exit
```

### Dato al ser emptydir un volume al vuelo no tiene lugar, se guarda en memoria.

3. Eliminar Pod y probar persistencia:

```console
$ kubectl delete -f pod_emptydir.yaml
$ kubectl apply -f pod_emptydir.yaml
$ kubectl exec -ti alpine-empty sh
# ls -la /scratch/
```

**************************************

## ConfigMaps:

1. Crear archivo configmap1.yaml con contenido:

```console
$ nano configmap1.yaml
```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
 name: nginx-cm
data:
 MY_KEY: "MY_VALUE"
 foo: bar
 special.how: very
 special.type: charm
```

```console
$ kubectl apply -f configmap1.yaml
$ kubectl get configmap
$ kubectl describe configmap nginx-cm
```

### Pod usando ConfigMap:

2. Crear archivo pod_cm.yaml con contenido:

```console
$ nano pod_cm.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: nginx-pod-cm
spec:
 containers:
 - name: nginx
   image: nginx:alpine
   env:
     - name: MY_KEY
       value: "MY_VALUE2"
     - name: foo
       valueFrom:
         configMapKeyRef:
           name: nginx-cm
           key: foo
     - name: SPECIAL_LEVEL_KEY
       valueFrom:
         configMapKeyRef:
           name: nginx-cm
           key: special.how
```

3. Ejecutar

```console
$ kubectl apply -f pod_cm.yaml 
$ kubectl get pods
$ kubectl exec -ti nginx-pod-cm sh
# echo $foo
# echo $SPECIAL_LEVEL_KEY
# env
# exit
 ```

4. Crear archivo pod_cm_all.yaml con contenido:

```console
$ nano pod_cm_all.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-pod-cm-all
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    envFrom:
      - configMapRef:
          name: nginx-cm
```

5. Ejecutar

```console
$ kubectl apply -f pod_cm_all.yaml
$ kubectl get pods
$ kubectl exec -ti nginx-pod-cm-all sh
```

**************************************

## ConfigMap as File:

1. Crear archivo configmap2.yaml

```console
$ nano configmap2.yaml
```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: myconfigdata
data:
  myfile.txt: |
    This is a file demo
    this is a second line

    this is a ...
```

2. Ejecutar

```console
$ kubectl apply -f configmap2.yaml
$ kubectl describe cm myconfigdata
```

### Pod usando ConfigMap as file:

1. Crear archivo pod_cm_file.yaml con contenido:

```console
$ nano pod_cm_file.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-pod-cm-file
spec:
 volumes:
   - name: myfile
     configMap:
       name: myconfigdata
 containers:
   - name: nginx
     image: nginx:alpine
     volumeMounts:
     - name: myfile
       mountPath: /mnt/files/
```

```console
$ kubectl apply -f pod_cm_file.yaml
$ kubectl get pods
$ kubectl exec -ti nginx-pod-cm-file sh
# ls /mnt/files/
# cat /mnt/files/myfile.txt
# exit
```

### Opcional: 

2. misma implementacion, pero como Deployment deploy_cm.yaml

```console
$ nano deploy_cm.yaml
```

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-dep
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: myfile
          configMap:
            name: myconfigdata
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 80
          volumeMounts:
            - name: myfile
              mountPath: /mnt/files/
```

```console
$ kubectl apply -f deploy_cm.yaml
$ kubectl exec -ti nginx-dep-788c7849c7-7h47k -- ls /mnt/files/
```

### Cuando los configmaps son actualizados, los pods no se actualizan automaticamente para ello se puede usar reloader -> https://github.com/mario21ic/k8s-demos/tree/master/reloader


**************************************

## K8s Secrets:

1. Crear archivo secret1.yaml

```console
$ nano secret1.yaml
```

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: my-secret
type: Opaque
data:
  hello: SGVsbG8gZnJvbSBBdWxhVXRpbA==
  key: T1VSX0FQSV9BQ0NFU1NfS0VZCg==
  token: U0VDUkVUXzd0NDgzNjM3OGVyd2RzZXIzNAo=
```

```console
$ kubectl apply -f secret1.yaml
$ kubectl get secret
$ kubectl describe secret my-secret
 ```

### Pod que consuma el secret:

2. Archivo pod_secret.yaml

```console
$ nano pod_secret.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-pod-secret
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    env:
      - name: HELLO
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: hello
```

```console
$ kubectl apply -f pod_secret.yaml
$ kubectl get pods
$ kubectl exec -ti nginx-pod-secret -- env
```

3. Dinamica: Recuperar los otros dos valores (key y token).

4. Limpiar cluster:

```console
$ kubectl delete all --all
```
