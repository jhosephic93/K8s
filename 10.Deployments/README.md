# DEPLOYMENTS

- Hace lo que ReplicaSet hace + Administra actualizaciones.
- Describe un estado deseado.
- Administra actualizaciones (Rolling Back)
  - Resultados de traducción despliegue controlado desde el estado actual al estado deseado.
- Guarda el historial de las versiones de los rolling back.

1. Crear archivo deployment.yaml

```console
$ nano deployment.yaml
```

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
 name: nginx-dp
spec:
 replicas: 2
 selector:
   matchLabels:
     app: nginx
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:alpinev1
```

2. Ejecutar

```console
$ kubectl apply -f deployment.yaml
$ kubectl get deployments
$ kubectl get pods
```

3. Listado de su replicaset:

```console
$ kubectl get replicaset
$ kubectl get rs
```

4. Scaling:

```
$ kubectl scale deployments nginx-dp --replicas=5
$ kubectl get pods
```

5. Describe

```
$ kubectl describe deployments nginx-dp
```

6.  Eliminar

```console
$ kubectl delete deployments nginx-dp
```

## Update del Deploy

1. Crear archivo rolling_update.yaml

```console
$ nano rolling_update.yaml
```

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
 name: nginx-dp-rl
spec:
 selector:
   matchLabels:
     app: nginx
 replicas: 5
 strategy:
   type: RollingUpdate
   rollingUpdate:
     maxSurge: 2
     maxUnavailable: 1
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:alpine
       #image: nginx:alpine-perl
```

2. Ejecutar.

```console
$ kubectl apply -f rolling_update.yaml
$ kubectl get deploy
```

3. Verificar version del docker image:

```console
$ kubectl get pods
$ kubectl describe pod <pod_id>
```
## Probando update

1. Descomentar la ultima linea del archivo rolling_update.yaml (image: mario21ic/nginx:alpinev2)

```console
$ kubectl apply -f rolling_update.yaml 
```

2. Revisar que se generen los nuevos pods:

```console
$ kubectl get pods
$ kubectl get deploy
```

3. Verificar version del docker image:

```console
$ kubectl describe pod <pod_id>
```

## Deployment history

1. Listado de revisiones

```console
$ kubectl rollout history deployments
```

### Status:

```console
$ kubectl rollout status deployments nginx-dp-rl
```

1. Revisar detalles de revision:

```console
$ kubectl rollout history deployments nginx-dp-rl --revision=2
$ kubectl rollout history deployments nginx-dp-rl --revision=1
```

### Rollback:

```console
$ kubectl rollout undo deployments nginx-dp-rl --to-revision=1
$ kubectl rollout history deployments
```

1. Mirar la nueva revision:

```console
$ kubectl rollout history deployments nginx-dp-rl --revision=3
```

2. Finalmente describe del deploy:

```console
$ kubectl describe deploy nginx-dp-rl
$ kubectl get pods
$ kubectl describe pods <pod-id>
```

3. Eliminar:

```console
$ kubectl delete deploy nginx-dp-rl
```