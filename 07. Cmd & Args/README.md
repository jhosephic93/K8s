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
mkdir image-entrypoint && cd image-entrypoint
docker run composer:latest --version 
docker history composer:latest 
docker run -ti --entrypoint=/bin/sh composer:latest #Ingresamos al container mediante bash
exit
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

- In the RESTARTS column it should say 5 -> It will say CrashLoopBackOff because Kubernetes, unlike Docker, does not accept entrypoint, since it is composer:latest, its entrypoint is composer, therefore that command will be executed but since it is not a command that is maintained/perennial then the container will restart.

1. Describe and obtain pod logs:

    ```console
    kubectl describe pod/pod-cmd-default
    kubectl logs -f pod-cmd-default #The logs will be the output of "composer"
    ```

## ADDING CMD

1. Create file pod-cmd-propio.yaml with content:

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

    - Summary: Kubernetes does NOT accept Entrypoints from Docker Images

## CORRECT WAY TO INVOKE COMANDS AND ARGS

1. Create file pod-cmd-args.yaml

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
    kubectl get pods #It will appear in STATUS Completed because "composer --version" was executed and since it is not a command that is maintained/perennial, the container is deleted.
    kubectl logs -f pod-composer
    ```

2. Delete all pods:

    ```console
    kubectl delete pods --all
    ```
