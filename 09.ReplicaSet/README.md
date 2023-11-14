# ReplicaSet

- Before ReplicationController was used, now ReplicaSet is used
- Ensures that the specified number of pod replicas are running.
  - Create and delete pods as needed.
- Selector + Pod template + Replica count (number of replicas).

1. Create file replicaset.yaml

    ```console
    nano replicaset.yaml
    ```

    ```yaml
    kind: ReplicaSet
    apiVersion: apps/v1
    metadata:
     name: myapp-rs
    spec:
     replicas: 5
     selector:
       matchLabels:
         app: myapp
     template:
       metadata:
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
    kubectl apply -f replicaset.yaml
    kubectl get replicaset
    kubectl describe replicaset myapp-rs
    kubectl get pods
    ```

3. Test that a new pod is generated if one is deleted:

    ```console
    kubectl delete pods <pod_id>
    ```

## Scale

1. Scale

    ```console
    kubectl scale rs myapp-rs --replicas=2
    kubectl get pods
    ```

## Delete rs

1. Delete rs

    ```console
    kubectl delete rs myapp-rs
    ```
