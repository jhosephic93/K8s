# ANNOTATIONS

- Key-value metadata
- Additional info on resources
- Don't impact resource behavior
- Useful for tools, libraries
- Store descriptions, usage, debugging
- Non-essential but informative data
- Flexible, modifiable anytime

1. Create file pods_annotations.yaml

    ```console
    nano pods_annotations.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: nginx-annotations
     annotations:
       commit: 44e3275e0
       logs: 'http://logservice.com/nginx'
       contact: 'Michaelt Inga <jhosephic93@gmail.com>'
    spec:
     containers:
     - name: nginx
       image: nginx:alpine
    ```

2. Execute.

    ```console
    kubectl apply -f pods_annotation.yaml
    kubectl describe pods nginx-annotations
    ```
