# KUBERNETES | PORT FORWARD

1. Create file pod-nginx.yaml with content:

    ```console
    nano pod-nginx.yaml
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

2. Execute

    ```console
    kubectl apply -f pod-nginx.yaml
    kubectl get pods
    kubectl port-forward pod/mynginx 8080:80
    ```

3. Execute in another terminal.

    ```console
    curl -I localhost:8080
    ```

   - The problem is that it's only for localhost, in case you need to expose the pod across the entire server.

    ```console
    kubectl port-forward --address 0.0.0.0 pod/mynginx 8080:80
    ```

    - Check in the browser http://ip-publica:8080/

4. En otro terminal revisar el log:

    ```console
    kubectl logs -f pod/mynginx
    ```

## KUBECTL PROXY OF THE CLUSTER

1. First terminal.

    ```console
    kubectl proxy
    ```

2. Second terminal | Accessing the 'mynginx' pod through port 80.

    ```console
    curl http://localhost:8001/api/v1/namespaces/default/pods/mynginx/proxy/
    curl http://localhost:8001/api/v1/namespaces/default/pods/<name-pod>/proxy/
    ```

3. Create file myapp-pod.yaml with content:

    ```console
    nano myapp-pod.yaml
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
    kubectl apply -f myapp-pod.yaml
    curl http://localhost:8001/api/v1/namespaces/default/pods/myapp-pod/proxy/
    ```

- WARNING: It will give an error since Kubectl proxy only works with those pods that expose port 80

## Exposing a Pod and editing index.html

1. On terminal:

    ```console
    kubectl port-forward --address 0.0.0.0 pod/mynginx 8080:80
    ```

2. Execute the command in another terminal.

    ```console
    $ kubectl exec -ti mynginx -- sh
    # vi /usr/share/nginx/html/index.html # Poner en español algun texto
    ```

    - Abrir en browser la ip publica de su nodo1, en mi caso: http://ip-publica:8080

3. Copy file from the Pod to my localhost and vice versa

    ```console
    $ kubectl cp mynginx:/usr/share/nginx/html/index.html index.html
    $ head index.html
    ##Edit index.html:
    $ nano index.html
    $ kubectl cp index.html mynginx:/usr/share/nginx/html/index.html
    ```

4. Option

    ```console
    kubectl exec -ti mynginx -- vi /usr/share/nginx/html/index.html
    ```

    - Abrir en browser la ip publica de su nodo1, en mi caso: http://ip-publica:8080

5. Revisar el consumo de Recursos de nodos:

    ```console
    minikube addons enable metrics-server
    minikube addons list|grep metric
    kubectl top nodes
    ```

    | NAME      | CPU(cores) |  CPU% |  MEMORY(bytes) |  MEMORY% |
    | :-------- |:----------:|:-----:|:--------------:|:--------:|
    | minikube  | 143m       |  3%   |  651Mi         |  8%      |

6. Nota: Herramienta para detectar puerto abierto

    ```console
    sudo apt install -y net-tools
    sudo netstat -lntp | grep 8080
    sudo netstat -lntp | grep 22
    ```
