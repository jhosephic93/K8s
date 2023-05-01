# Init Containers:

- Ejecutar antes de que se inicien los contenedores de la aplicación.
- Los contenedores de inicio son exactamente iguales a los contenedores regulares, excepto:
- Los contenedores de inicio siempre se ejecutan hasta su finalización.
- Cada contenedor de inicialización debe completarse correctamente antes de que comience el siguiente.
- Admite todos los campos y funciones de los contenedores de aplicaciones, incluidos los límites de recursos, los volúmenes y la configuración de seguridad.
- Además, los contenedores de inicio no son compatibles con las pruebas de preparación porque deben ejecutarse hasta el final antes de que el pod pueda estar listo.

```console

```
## LET'S GO

1. Crear pod_init.yaml con contenido:

```console
$ nano pod_init.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-init
  labels:
    app: nginx
spec:
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
```

2. Ejecutar

```console
$ kubectl apply -f pod_init.yaml
$ kubectl get pods
```

3. Debe salir mas o menos asi:

| NAME       | READY | STATUS   | RESTARTS | AGE |
| :--------- |:-----:| :-------:| :-------:|----:|
| nginx-init | 0/1   | Init:0/2 | 0        | 2m3s| 

```console
$ kubectl describe pod nginx-init
```

4. Revisar el log

```console
$ kubectl logs nginx-init # debe dar error
$ kubectl logs nginx-init -c init-myservice
```

5. Debe dar el siguiente error:

```
nslookup: can't resolve 'myservice'
waiting for myservice
```

## Pasar el 1er init:

1. Crear archivo myservice.yaml

```console
$ nano myservice.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

2. Aplicar y verificar:

```console
$ kubectl apply -f myservice.yaml
$ kubectl get service
$ kubectl get pods
```

3. Debe salir mas o menos:

| NAME       | READY | STATUS   | RESTARTS | AGE |
| :--------- |:-----:| :-------:| :-------:|----:|
| nginx-init | 0/1   | **Init:1/2** | 0        | 2m3s| 

```console
$ kubectl describe pod nginx-init
```

4. Debe salir en init-myservice:

| Ready: | True |
| :----- | ----:|

```console
$ kubectl logs nginx-init -c init-myservice
``` 

5. Debe salir mas o menos:

```console
Address 1: 10.100.230.8 myservice.default.svc.cluster.local
```

## Pasar el 2do init:

1. Crear archivo mydb.yaml con contenido:

```console
$ nano mydb.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

2. Aplicar y verificar:

```console
$ kubectl apply -f mydb.yaml
$ kubectl get service
$ kubectl describe pod nginx-init
$ kubectl logs nginx-init -c init-mydb
$ kubectl get pods
```

3. El pod debe estar en status Running debido a que ya se resolvieron los dos services del init.
