# INIT CONTAINERS

- Run before the application containers are started.
- Starter containers are exactly the same as regular containers, except:
  - Startup containers always run to completion.
  - Each initialization container must complete successfully before the next one begins.
  - Supports all fields and features of application containers, including resource limits, volumes, and security settings.
  - Additionally, startup containers do not support readiness tests because they must be run to completion before the pod can be ready.

## LET'S GO

1. Create file pod_init.yaml with content:

    ```console
    nano pod_init.yaml
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

2. Execute

    ```console
    kubectl apply -f pod_init.yaml
    kubectl get pods
    ```

3. It should come out more or less like this:

    | NAME       | READY | STATUS   | RESTARTS | AGE |
    | :--------- |:-----:| :-------:| :-------:|----:|
    | nginx-init | 0/1   | Init:0/2 | 0        | 2m3s|

    ```console
    kubectl describe pod nginx-init
    ```

4. Check log

    ```console
    kubectl logs nginx-init # debe dar error
    kubectl logs nginx-init -c init-myservice
    ```

5. It should give the following error:

    ```bash
    nslookup: can't resolve 'myservice'
    waiting for myservice
    ```

## Pass the 1st init

1. Create file myservice.yaml

    ```console
    nano myservice.yaml
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

2. Apply and check:

    ```console
    kubectl apply -f myservice.yaml
    kubectl get service
    kubectl get pods
    ```

3. It should come out more or less:

    | NAME       | READY | STATUS       | RESTARTS | AGE |
    | :--------- |:-----:| :-----------:| :-------:|----:|
    | nginx-init | 0/1   | **Init:1/2** | 0        | 2m3s|

    ```console
    kubectl describe pod nginx-init
    ```

4. We must see init-myservice:

    | Ready: | True |
    | :----- | ----:|

    ```console
    kubectl logs nginx-init -c init-myservice
    ```

5. Like this

    ```console
    Address 1: 10.100.230.8 myservice.default.svc.cluster.local
    ```

## Pass 2nd init

1. Create mydb.yaml with content:

    ```console
    nano mydb.yaml
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

2. Apply and check:

    ```console
    kubectl apply -f mydb.yaml
    kubectl get service
    kubectl describe pod nginx-init
    kubectl logs nginx-init -c init-mydb
    kubectl get pods
    ```

3. The pod must be in Running status because the two init services have already been resolved.
