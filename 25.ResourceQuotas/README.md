# Resource Quotas

- Manage limits at the memory and cpu level and limit the number of pods per namespace.
- To restrict the amount of resources consumed by each namespace.

    ```console
    kubectl delete namespace my-namespace
    kubectl create namespace my-namespace
    ```

1. Create file resource-quota.yaml

    ```console
    nano resource-quota.yaml
    ```

    ```yaml
    kind: ResourceQuota
    apiVersion: v1
    metadata:
     name: my-resource-quota
     namespace: my-namespace
    spec:
     hard:
       pods: 5
       "requests.cpu": "2"
       "requests.memory": 1024Mi
       "limits.cpu": "4"
       "limits.memory": 2048Mi
    ```

    ```console
    kubectl apply -f resource-quota.yaml
    kubectl get resourcequota -n my-namespace
    my-resource-quota   13s   pods: 0/5, requests.cpu: 0/2, requests.memory: 0/1Gi   limits.    cpu: 0/4, limits.memory: 0/2Gi
    ```

## Pod Illegal

1. Create file pod-illegal.yaml

    ```console
    nano pod-illegal.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: pod-illegal
      namespace: my-namespace
    spec:
      containers:
        - name: nginx-illegal
          image: nginx:alpine
    ```

2. Execute

    ```console
    kubectl apply -f pod-illegal.yaml #Error from server (Forbidden): error when creating     "pod-illegal.yaml": pods "pod-illegal" is forbidden: failed quota: my-resource-quota:     must specify limits.cpu,limits.memory,requests.cpu,requests.memory
    ```

## Pod with request and limits

1. Create file pod-legal.yaml

    ```console
    nano pod-legal.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: pod-one
     namespace: my-namespace
    spec:
     containers:
       - name: nginx-pod-one
         image: nginx:alpine
         resources:
           requests:
             memory: 768Mi
             cpu: "0.5"
           limits:
             memory: 1024Mi
             cpu: "2"
    ```

    ```console
    kubectl apply -f pod-legal.yaml
    kubectl get pods -n my-namespace
    kubectl get resourcequota -n my-namespace

    my-resource-quota   4m43s   pods: 1/5, requests.cpu: 500m/2, requests.memory: 768Mi/    1Gi   limits.cpu: 2/4, limits.memory: 1Gi/2Gi
    ```

## Pod legal 2

1. Create file pod-legal2.yaml

    ```console
    nano pod-legal2.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: pod-two
     namespace: my-namespace
    spec:
     containers:
       - name: nginx-pod-two
         image: nginx:alpine
         resources:
           requests:
             memory: 512Mi
             cpu: "0.5"
           limits:
             memory: 1024Mi
             cpu: "2"
    ```

    ```console
    kubectl apply -f pod-legal2.yaml # Debe salir error, porque 768+512 superan los 1024.
    ```

## Deploy legal

1. Create file deploy-legal.yaml

    ```console
    nano deploy-legal.yaml
    ```

    ```yaml
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: nginx-dp
      namespace: my-namespace
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 5
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:alpine
            resources:
              limits:
                memory: 0Mi
                cpu: "0"
    ```

2. Execute.

    ```console
    kubectl apply -f deploy-legal.yaml # ## Only 4 pods will be created since the resource quota limit is 5 pods (4 deploy + pod-one)
    kubectl describe resourcequota my-resource-quota -n my-namespace
    kubectl describe resourcequota my-resource-quota -n my-namespace
    kubectl get pods -n my-namespace
    kubectl get deploy -n my-namespace
    ```
