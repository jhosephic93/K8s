# ReplicaSet

- Antes se usaba ReplicationController ahora se usa ReplicaSet
- Garantiza que se está ejecutando el número especificado de réplicas de pod.
  - Crea y elimina pods segun sea necesario.
- Selector + Pod template + Replica count (numero de replicas).

1. Crear archivo replicaset.yaml

```console
$ nano replicaset.yaml
```

```yaml
kind: ReplicaSet
apiVersion: apps/v1
metadata:
 name: myapp-rs
spec:
 replicas: 5
 selector:
   matchLabels:
     app: myapp
 template:
   metadata:
     labels:
       app: myapp
   spec:
     containers:
     - name: myapp-container
       image: alpine
       command: ['sh', '-c', 'echo Hello Aulautil! && sleep 3600']
```

2. Ejecutar

```console
$ kubectl apply -f replicaset.yaml
$ kubectl get replicaset
$ kubectl describe replicaset myapp-rs
$ kubectl get pods
```

3. Probar que se genere un nuevo pod si se elimina alguno:

```console
$ kubectl delete pods <pod_id>
```

## Scale:

```console
$ kubectl scale rs myapp-rs --replicas=2
$ kubectl get pods
```

## Eliminar:

```console
$ kubectl delete rs myapp-rs
```