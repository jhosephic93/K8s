# DEPLOY TEST POD

## EXAMPLE 01

1. Create file.

   ```console
   nano pod1.yaml
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

2. Execute

    ```console
    kubectl apply -f pod1.yaml
    kubectl get pods #Listar los pods
    kubectl logs myapp-pod #Revisar logs del pod:
    kubectl describe pods myapp-pod #Describir Pods
    ```

3. Execute a shell inside the pod.

    ```console
    $ kubectl exec -ti myapp-pod -- sh
    # id
    # ps aux
    # env
    ```

4. Analyze the output of the env command in relation to the Kubernetes variables. (hostname)
Install htop and curl, then run htop and check processes.

    ```console
    # apk add htop curl
    # htop
    # exit
    ```

5. Retrieve the pod information in yaml format.

    ```console
    kubectl get pod myapp-pod -o yaml
    ```

## EXAMPLE02

1. Create file.

    ```console
    pod-nginx.yaml
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

2. Execute

    ```console
    kubectl apply -f pod-nginx.yaml
    kubectl get pod
    kubectl logs -f mynginx
    kubectl describe pod mynginx
    ```

3. Get the IP of the Pod and perform a curl inside the minikube.

    ```console
    kubectl describe pod mynginx | grep IP
    minikube ssh
    curl -I <ip-privada-pod>
    exit
    ```

4. Checking inside the nginx pod:

    ```console
    $ kubectl exec -ti mynginx -- sh
    # ps aux
    # env
    # curl -I localhost
    ```

5. Delete the main process and check the pod listing in the restart column.

    ```console
    $ kubectl exec -ti mynginx -- sh
    # kill 1
    $ watch kubectl get pods
    ```

- Exercise: terminate the main process until the status shows 'CrashLoopBackOff'.
