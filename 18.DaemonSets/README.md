# DaemonSets

- Run a replica on every node in the cluster.
- Examples of use: monitoring, logging, etc
- Are similar to Deployments, because both create Pods.
- Alternatives to DaemonSet:
  - Init Scripts
  - Bare Pods
  - Static Pods

1. Create file nginx-daemonset.yaml

    ```console
    nano nginx-daemonset.yaml
    ```

    ```yaml
    kind: DaemonSet
    apiVersion: apps/v1
    metadata:
      name: nginx-ds
    spec:
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          tolerations:
          - key: node-role.kubernetes.io/master
            effect: NoSchedule
          containers:
          - name: nginx
            image: nginx:alpine
          terminationGracePeriodSeconds: 30
    ```

    ```console
    kubectl apply -f nginx-daemonset.yaml
    kubectl get daemonset
    ```

2. Check details and pods:

    ```console
    kubectl describe daemonset nginx-ds
    kubectl get pods
    ```

3. List all the daemonset of the system:

    ```console
    kubectl get daemonset -n kube-system
    kubectl get daemonset -A
    ```

## DaemonSet with NodeSelector

1. Set labels to a node:

    ```console
    kubectl get node
    kubectl label node minikube ssd=true
    kubectl get node --show-labels
    ```

2. Create file nginx-daemonset2.yaml

    ```console
    nano nginx-daemonset2.yaml
    ```

    ```yaml
    kind: DaemonSet
    apiVersion: apps/v1
    metadata:
      name: nginx-ds-node
    spec:
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          nodeSelector:
            ssd: "true"
          tolerations:
          - key: node-role.kubernetes.io/master
            effect: NoSchedule
          containers:
          - name: nginx
            image: nginx:alpine
          terminationGracePeriodSeconds: 30
    ```

    ```console
    kubectl apply -f nginx-daemonset2.yaml
    kubectl get daemonset # Debe aparecer ssd=true en la columna Node Selector.
    ```
