# Desplegar POD de prueba.

# EXAMPLE01

1. Crear archivo.

```console
$ nano pod1.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
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
$ kubectl apply -f pod1.yaml
$ kubectl get pods #Listar los pods
$ kubectl logs myapp-pod #Revisar logs del pod:
$ kubectl describe pods myapp-pod #Describir Pods
```

3. Ejecutar un sh dentro del pod

```console
$ kubectl exec -ti myapp-pod -- sh
# id
# ps aux
# env
```

4. Analizar la salida del comando env respecto a las variables de K8s. (hostname)
Instalar htop y curl, luego ejecutar htop y revisar procesos.

```console
# apk add htop curl
# htop
# exit
```

4. Recuperar la informacion del pod en formato yaml:

```console
$ kubectl get pod myapp-pod -o yaml
```

# EXAMPLE02

1. Crear archivo.

```console
$ pod-nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mynginx
  labels:
    app: mynginx
spec:
  containers:
  - name: mynginx-container
    image: nginx:alpine
```

2. Ejecutar

```console
$ kubectl apply -f pod-nginx.yaml
$ kubectl get pod
$ kubectl logs -f mynginx
$ kubectl describe pod mynginx
```

3. Obtener la IP del Pod y hacer curl dentro del minikube:

```console
$ kubectl describe pod mynginx | grep IP
$ minikube ssh
$ curl -I <ip-privada-pod>
$ exit
```

4. Revisando el interior del pod nginx:

```console
$ kubectl exec -ti mynginx -- sh
# ps aux
# env
# curl -I localhost
```

5. Eliminar proceso principal y revisar el listado de pods columna restart:

```console
$ kubectl exec -ti mynginx -- sh
# kill 1
$ watch kubectl get pods
```
- Ejercicio: eliminar el proceso principal hasta que el status te marque “CrashLoopBackOff”