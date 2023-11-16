# Namespaces

1. List all namespaces

    ```console
    kubectl get namespaces
    kubectl get pod 
    kubectl get pod -n default
    ```

2. List all resources of a namespace

    ```console
    kubectl get pod -n kube-system
    kubectl get svc -n kube-system
    kubectl get deploy -n kube-system
    kubectl get all -n kube-system
    ```

3. Create file my-namespaces.yaml

    ```console
    nano my-namespaces.yaml
    ```

    ```yaml
    kind: Namespace
    apiVersion: v1
    metadata:
      name: red-team
    ---
    kind: Namespace
    apiVersion: v1
    metadata:
      name: green-team

    Aplicar y listar:
    $ kubectl apply -f my-namespaces.yaml
    $ kubectl get namespaces

    Especificando namespace:
    Crear archivo pod_namespace1.yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-pod
      namespace: red-team
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
    ```

    ```console
    kubectl apply -f pod_namespace1.yaml
    kubectl get pods -n red-team
    ```

## Applying manifest in a specific namespace

1. Create file pod_namespace2.yaml

    ```console
    nano pod_namespace2.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
    $ kubectl apply -f pod_namespace2.yaml -n green-team
    $ kubectl get pods -n green-team

    Probar recursos aislados:
    Crear archivo service_namespace.yaml
    kind: Service
    apiVersion: v1
    metadata:
     name: nginx-svc
    spec:
     ports:
     - port: 80
     ```

2. Execute

     ```console
    kubectl apply -f service_namespace.yaml -n red-team
    kubectl exec -ti nginx-pod -n green-team -- sh
    ping nginx-svc #Debe salir error porque el service es del red-team namespace
    kubectl exec -ti nginx-pod -n red-team – sh
    ping nginx-svc #Debe funcionar porque el service esta en el red-team namespace
    ```

3. List resources:

    ```console
    kubectl get pods -n red-team
    kubectl get pods -n green-team
    kubectl get svc -n red-team
    $ kubectl get svc -n green-team
    ```
