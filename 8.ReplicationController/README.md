# REPLICATION CONTROLLER:

- Se encarga de desplegar y mantener el numero de replicas.
- Antes se usaba ReplicationController ahora se usa ReplicaSet.
- Reasons:
  - Redundancy: Para tener alta disponibilidad.
  - Scale: Escalar pods.
  - Sharding: Repartir el procesamiento de computo en diferentes unidades o instancias.


1. Crear archivo replicationcontroller.yaml con contenido:

```console
$ nano replicationcontroller.yaml
```

```yaml
kind: ReplicationController
apiVersion: v1
metadata:
 name: myapp-rp
spec:
 replicas: 3
 selector:
   app: myapp
 template:
   metadata:
     name: myapp
     labels:
       app: myapp
   spec:
     containers:
     - name: myapp
       image: nginx:alpine
       resources:
         limits:
           cpu: 0.5
```

2. Ejecutar

```
$ kubectl apply -f replicationcontroller.yaml
$ kubectl get replicationcontroller
$ kubectl get rc
$ kubectl get pods
```

3. Eliminar todos los pods:

```console
$ kubectl delete pods --all
$ kubectl get pods #Verificar que haya generado 3 nuevos pod
```

4. Eliminar replicationcontroller

```console
$ kubectl delete rc myapp-rp
```

## Scaling:

```console
$ kubectl scale rc myapp-rp --replicas=6
$ kubectl get pods
```

## Eliminar:

```console
$ kubectl delete rc myapp-rp
```