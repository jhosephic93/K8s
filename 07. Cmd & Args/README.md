# POD CMD/ARGS VS DOCKER ENTRYPOINT

## K8s does not have Entrypoint, but it does have commands and arguments

1. Create pod-cmd-args.yaml file with content.

    ```console
    nano pod-cmd-args.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: pod-cmd-args
    spec:
      containers:
      - name: alpine
        image: nginx:alpine
        command: ["/bin/sh"]
        args: ["-c", "while true; do date; sleep 5; done"]
    ```

    ```console
    kubectl apply -f pod-cmd-args.yaml
    kubectl get pods
    kubectl logs -f pod-cmd-args
    ```

2. Run sh and see processes:

    ```console
    kubectl exec -ti pod-cmd-args -- sh
    ps aux  #All the cmd and args in a single line | We see the cmd that I put
    ```

3. Validar cual es el CMD por default del nginx:alpine:

    ```console
    minikube ip
    minikube ssh
    docker history nginx:alpine # We see the nginx cmd that was not executed because I sent it my own cmd.
    exit
    ```

## Docker Entrypoint vs Kubernetes

### DOCKER ENTRYPOINT

```console
$ mkdir image-entrypoint && cd image-entrypoint
$ docker run composer:latest --version 
$ docker history composer:latest 
$ docker run -ti --entrypoint=/bin/sh composer:latest #Ingresamos al container mediante bash
# exit
```

### WITH KUBERNETES

```console
nano pod-cmd-default.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cmd-default
  labels:
    app: composer
spec:
  containers:
  - name: cmd-default-container
    image: composer:latest
```

```console
kubectl apply -f pod-cmd-default.yaml
kubectl get pods
```

- En la columna RESTARTS debe decir 5 -> Dira CrashLoopBackOff porque Kubernetes a diferencia de Docker no acepta entrypoint, dado que es composer:latest su entrypoint es composer por ende se ejecutará ese comando pero al no ser un comando que se mantiene/perene entonces el container se reiniciara.

1. Describir y obtener logs del pod:

    ```console
    kubectl describe pod/pod-cmd-default
    kubectl logs -f pod-cmd-default #Los logs sera el output de "composer"
    ```

## AGREGANDO CMD

1. Archivo pod-cmd-propio.yaml con contenido:

    ```console
    nano pod-cmd-propio.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-cmd-propio
      labels:
        app: composer
    spec:
      containers:
      - name: cmd-propio-container
        image: composer:latest
        command: ["--version"]
    ```

2. Ejecutar

    ```console
    kubectl apply -f pod-cmd-propio.yaml
    kubectl get pods
    kubectl logs -f pod-cmd-propio
    kubectl describe pod pod-cmd-propio
    ```

    - Resumen: Kubernetes NO acepta Entrypoints de Docker Images

## FORMA CORRECTA DE INVOCAR COMANDS Y ARGS

1. Crear archivo pod-cmd-args.yaml con contenido:

    ```console
    nano pod-cmd-args.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-composer
      labels:
        app: composer
    spec:
      containers:
      - name: cmd-composer-container
        image: composer:latest
        command: ["composer"]
        args: ["--version"]
    ```

    ```console
    kubectl apply -f pod-cmd-args.yaml
    kubectl get pods #Aparecera en STATUS Completed porque se ejecuto "composer --version" y    al no ser un comando que se mantiene/perene el container se elimina.
    kubectl logs -f pod-composer
    ```

2. Eliminar todos los pods:

    ```console
    kubectl delete pods --all
    ```
