# Readiness:

### Para probar si el pod esta saludable y Ready. 

1. Crear el archivo deploy_readiness_liveness.yaml como sigue:

```console
$ nano deploy_readiness_liveness.yaml
```

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
 name: nginx-dp-liveness
spec:
 selector:
   matchLabels:
     app: nginx
 replicas: 2
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:alpine
       readinessProbe:
         httpGet:
           path: /ready
           port: 80
         periodSeconds: 2
         initialDelaySeconds: 0
         failureThreshold: 3
         successThreshold: 1
       ports:
         - name: web
           containerPort: 80
           protocol: TCP
```

2. Aplicar nuevo cambio y listar:

```console
$ kubectl apply -f deploy_readiness_liveness.yaml
$ kubectl get pods # La columna “status” debe estar en “running” pero la columna “ready” debe estar en “0/1”
```

```console
$ kubectl describe pod <pod-id> # Debe aparecer algo asi: Readiness probe failed: HTTP probe failed with statuscode: 404
```

3. Probar ruta /ready

```console
$ kubectl exec -ti nginx-dp-liveness-8cf79d5c7-r9rff -- sh
# apk add curl
# curl localhost/ready #Sale error porque no existe la ruta
# curl localhost/
# exit
```

## Probar con Service

1. Que este funcionando creamos un service_readiness_liveness.yaml con contenido:

```console
$ nano service_readiness_liveness.yaml
```

```yaml
kind: Service
apiVersion: v1
metadata:
 name: nginx-np-readiness-liveness
spec:
 type: NodePort
 selector:
   app: nginx
 ports:
   - name: web
     port: 80 # service interno
     targetPort: 80 # container
```

2. Ejecutar

```console
$ kubectl apply -f service_readiness_liveness.yaml
$ kubectl get svc
$ kubectl describe service/nginx-np-readiness-liveness #Debe estar en blanco la seccion de Endpoints
$ curl $(minikube ip):32002		# Sale conexion refused
```

3. Editamos el archivo deploy_readiness_liveness.yaml dejando la linea como sigue:
path: /

```console
$ nano deploy_readiness_liveness.yaml
```

4. Y ejecutar nuevamente:

```
$ kubectl apply -f deploy.yaml
$ kubectl get pods
$ kubectl describe service/nginx-np-readiness-liveness
```
5. Ahora si debe encontrar a los 2 pods saludables similar a:

```vim
Endpoints:                172.17.0.4:80,172.17.0.6:80
```

**********************************

# Liveness:

1. Para poder reiniciar un pod. Modificar el archivo deploy_readiness_liveness.yaml

```console
$ nano deploy_readiness_liveness.yaml
```

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
 name: nginx-dp-liveness
spec:
 selector:
   matchLabels:
     app: nginx
 replicas: 2
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: mario21ic/nginx:alpinev1
       readinessProbe:
         httpGet:
           path: /
           port: 80
         initialDelaySeconds: 0
         periodSeconds: 2
         failureThreshold: 3
         successThreshold: 1
       livenessProbe:
         httpGet:
           path: /live
           port: 80
         initialDelaySeconds: 5
         timeoutSeconds: 1
         periodSeconds: 10
         failureThreshold: 3
       ports:
         - name: web
           containerPort: 80
           protocol: TCP
```

2. Ejecutar

```console
$ kubectl apply -f deploy_readiness_liveness.yaml
$ kubectl get deploy
$ kubectl get pods #Debe salir que esta corriendo, pero tiene varios “restarts”
```

3. Para probar entrar en uno de los pods:

```console
$ kubectl exec -ti <pod-id> -- sh  #Pod sera terminado por liveness
# apk add curl
# curl localhost/live
```

4. Para ver los motivos

```console
$ kubectl describe pod #Debe salir error:
```

5. Liveness probe failed: HTTP probe failed with statuscode: 404

```console
$ kubectl describe service/nginx-np-readiness-liveness
```

6. Fixeamos el archivo deploy_readiness_liveness.yaml dejando la linea como sigue:
path: /

```console
$ nano deploy_readiness_liveness.yaml
```

7. Y ejecutar nuevamente:

```console
$ kubectl apply -f deploy_liveness.yaml
$ kubectl get pods
```

8. Deben estar eliminandose los dos anteriores y generando 2 nuevos pods y probar de nuevo.

```console
$ kubectl describe svc nginx-np-readiness-liveness
$ curl $(minikube service nginx-np-readiness-liveness --url)
```