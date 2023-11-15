# Environments

1. Create file pod_env.yaml

    ```console
    nano pod_env.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-env
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        env:
          - name: MY_KEY
            value: "MY_VALUE"
          - name: foo
            value: "bar"
    ```

    ```console
    kubectl apply -f pod_env.yaml
    kubectl get pods
    kubectl exec -ti nginx-env sh
    echo $MY_KEY
    echo $foo
    env
    exit
    ```
