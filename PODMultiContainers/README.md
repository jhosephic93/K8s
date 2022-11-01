# POD Multi Containers

1. Crear archivo pod-multi-container.yaml con contenido

```console
$ nano pod-multi-container.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: multi-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
      name: http
      protocol: TCP
  - name: alpine
    image: alpine:latest
    command: ["/bin/sh"]
    args: ["-c", "while true; do date; sleep 5; done"]
```

2. Ejecutar

```console
$ kubectl apply -f pod-multi-container.yaml
$ kubectl get pods multi-pod
```

3. Entrar a los containers y **VERIFICAMOS QUE TENGAN LA MISMA IP**:

```console
$ kubectl exec -ti multi-pod -c nginx -- sh
# ps aux
```

- Veremos que comparte la misma red que alpine

```console
# ifconfig
# exit
```

```console
$ kubectl exec -ti multi-pod -c alpine -- sh
# ps aux
# ifconfig
# apk add curl
# curl -I localhost
```
