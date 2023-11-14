# Limit CPU and Mem Usage

## Limit CPU

- Tip: It is recommended to only use **request** in Pods, and not have **Limits**
- More Info -> [Stop-Using-CPU-Limits](https://home.robusta.dev/blog/stop-using-cpu-limits)

1. Create pod-cpu.yaml file with content:

    ```console
    nano pod-cpu.yaml
    ```

    ```yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-cpu-demo
    spec:
      containers:
      - name: pod-cpu-demo-ctr
        image: vish/stress
        args:
        - --cpus
        - "2"
        resources:
          requests:
            cpu: "0.5"
          limits:
            cpu: "1"
    ```

2. Execute:

    ```console
    kubectl apply -f pod-cpu.yaml
    kubectl get pods
    kubectl describe pods/pod-cpu-demo #Inspeccionar
    ```

3. Check consumption, first verify that the metric server addon is enabled:

    ```console
    minikube addons list|grep metric
    minikube addons enable metrics-server
    kubectl top pods pod-cpu-demo
    ```

4. Logs:

    ```console
    kubectl logs -f pods/pod-cpu-demo
    sudo htop  #Revisar consumo de cpu:
    ```

## Setting CPU 5

1. Create pod_cpu2.yaml file with content:

    ```console
    nano pod_cpu2.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: pod-cpu-demo-max
    spec:
     containers:
     - name: pod-cpu-demo-ctr
       image: vish/stress
       args:
         - -cpus
         - "5"
       resources:
         limits:
           cpu: "5"
    ```

2. Apply:

    ```console
    kubectl apply -f pod_cpu2.yaml
    kubectl get pods
    ```

3. Inspect - We will see what happens to pending | We'll see what it says are not available.

    ```console
    kubectl describe pods pod-cpu-demo-max
    ```

4. Check consumption in the Limits node says 1(25%) for the pod-cpu-demo

    ```console
    kubectl describe node
    ```

5. Edit pod_cpu2.yaml to leave only 2 cpu units as follows:

    ```yaml
    cpu: "2"
    ```

    ```console
    kubectl delete -f pod_cpu2.yaml
    kubectl apply -f pod_cpu2.yaml
    kubectl top nodes #Verificar consumo en nodo:
    kubectl describe nodes
    ```

6. Verify that the **CPU REQUEST IS THE SAME AS THE LIMIT**, because it was not defined:

    ```console
    kubectl describe pods pod-cpu-demo-max
    ```

## Exceed CPU usage

1. Delete Pods

    ```console
    kubectl delete pods --all
    kubectl describe nodes
    ```

2. Create file with pod_cpu3.yaml content:

    ```console
    nano pod_cpu3.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: pod-cpu-max
    spec:
      containers:
      - name: pod-cpu-max
        image: ubuntu:18.04
        command: ["/bin/sh"]
        args: ["-c", "while true; do date; sleep 5; done"]
        resources:
          limits:
            cpu: "2"
    ```

    ```console
    kubectl apply -f pod_cpu3.yaml
    ```

3. Check the resources used in the node's CPU:

    ```console
    kubectl describe nodes
    ```

4. It should come out more or less like this:

    | Resource  | Requests    | Limits  |
    | :-------- |:-----------:| -------:|
    | cpu       | 2750m (68%) | 2 (50%) |

5. Install stress and try to use the assigned cpus:

    ```console
    kubectl exec -ti pod-cpu-max -- bash
    apt update
    apt install -y stress
    stress --cpu 2
    ```

6. Then change to --cpu 4

    ```console
    stress --cpu 4
    ```

7. In another terminal check CPU usage:

    ```console
    sudo htop
    kubectl top pods
    ```

## RESOURCE MEMORY | Limit MEM

1. Create pod_mem.yaml file with content:

    ```console
    nano pod_mem.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: pod-memory-demo
    spec:
     containers:
     - name: memory-demo-ctr
       image: polinux/stress
       resources:
         limits:
           memory: "200Mi"
         requests:
           memory: "100Mi"
       command: ["stress"]
       args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
    ```

2. Execute

    ```console
    kubectl apply -f pod_mem.yaml
    kubectl get pods
    kubectl describe pods pod-memory-demo
    ```

3. Check memory usage:

    ```console
    kubectl top pods
    kubectl top nodes
    kubectl describe nodes
    ```

4. Inspect:

    ```console
    kubectl exec -ti pod-memory-demo -- sh
    free -m
    ps aux
    exit
    ```

## Exceed Memory

1. Create file pod_mem2.yaml with:

    ```console
    nano pod_mem2.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: pod-memory-demo-max
    spec:
     containers:
     - name: memory-demo-ctr
       image: polinux/stress
       resources:
         limits:
           memory: "200Mi"
       command: ["stress"]
       args: ["--vm", "1", "--vm-bytes", "350M", "--vm-hang", "1"]
    ```

2. Apply:

    ```console
    kubectl apply -f pod_mem2.yaml
    kubectl describe pods pod-memory-demo-max
    ```

3. Review consumption and behavior:

    ```console
    watch kubectl top pods
    ```
