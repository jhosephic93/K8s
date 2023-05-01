# KUBERNETES | FORWARD DE PUERTOS:

1. Crear el archivo pod-nginx.yaml con contenido:

```console
$ nano pod-nginx.yaml
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
    ports:
    - containerPort: 80
      protocol: TCP
```
2. Ejecutar

```console
$ kubectl apply -f pod-nginx.yaml
$ kubectl get pods
$ kubectl port-forward pod/mynginx 8080:80
```

3. En otro terminal ejecutar:

```console
$ curl -I localhost:8080
```

- El problema es que solo es para localhost, en caso de necesitar exponer el pod en todo el server:

```console
$ kubectl port-forward --address 0.0.0.0 pod/mynginx 8080:80
```

- Revisar en el browser http://ip-publica:8080/

4. En otro terminal revisar el log:

```console
$ kubectl logs -f pod/mynginx
```


# KUBECTL PROXY DEL CLUSTER

1. Primer terminal.

```console
$ kubectl proxy
```

2. Segundo terminal | Accediendo al pod "mynginx" por el puerto 80

```console
$ curl http://localhost:8001/api/v1/namespaces/default/pods/mynginx/proxy/
$ curl http://localhost:8001/api/v1/namespaces/default/pods/<name-pod>/proxy/
```

3. Crear archivo myapp-pod.yaml con contenido:

```console
$ nano myapp-pod.yaml
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

```console
$ kubectl apply -f myapp-pod.yaml
$ curl http://localhost:8001/api/v1/namespaces/default/pods/myapp-pod/proxy/
```

### WARNING: Nos dara error dado que Kubectl proxy solo funciona con aquellos pods que exponen el puerto 80.

## Exponiendo un Pod y editando index.html

4. En un terminal:

```console
$ kubectl port-forward --address 0.0.0.0 pod/mynginx 8080:80
```

5. En otro terminal ejecutar el comando:

```console
$ kubectl exec -ti mynginx -- sh
# vi /usr/share/nginx/html/index.html # Poner en español algun texto
```

- Abrir en browser la ip publica de su nodo1, en mi caso: http://ip-publica:8080 

6. Copiar archivo desde el Pod hacia mi localhost y viceversa:

```console
$ kubectl cp mynginx:/usr/share/nginx/html/index.html index.html
$ head index.html
##Editar su index.html:
$ nano index.html
$ kubectl cp index.html mynginx:/usr/share/nginx/html/index.html
```

7. Opcion

```console
$ kubectl exec -ti mynginx -- vi /usr/share/nginx/html/index.html
```

- Abrir en browser la ip publica de su nodo1, en mi caso: http://ip-publica:8080

8. Revisar el consumo de Recursos de nodos:

```console
$ minikube addons enable metrics-server
$ minikube addons list|grep metric
$ kubectl top nodes
```

| NAME      | CPU(cores) |  CPU% |  MEMORY(bytes) |  MEMORY% |
| :-------- |:----------:|:-----:|:--------------:|:--------:|
| minikube  | 143m       |  3%   |  651Mi         |  8%      |


9. Nota: Herramienta para detectar puerto abierto

```console
$ sudo apt install -y net-tools
$ sudo netstat -lntp | grep 8080
$ sudo netstat -lntp | grep 22
```