# Pod Autoscaling | HPA - Horizontal Pods AutoScaling

- Two types
  - Vertical = Modify the size of an EC2
  - Horizontal
    - Horizontal pod Scaling -> $ kubectl scale deploy nginx-dp --replicas=3
- K8s allows you to define a simple policy to perform automatic HPA (Horizontal Scaling Pods).
- The metric used for this is CPU consumption, custom metrics are allowed, but each installation is responsible for generating them, since there is not yet a defined standard.
- Heapster/Metricserver must be activated for it to work correctly.
- Review the controller-manager configuration for metric details.

1. Clone repository:

    ```console
    git clone https://github.com/mario21ic/kubernetes-hpa-example-minikube.git
    cd kubernetes-hpa-example-minikube
    ```

2. Create namespace:

    ```console
    kubectl create namespace itsmetommy
    kubectl get namespace
    ```

3. Create deployment:

    ```console
    kubectl apply -f deployment.yaml
    kubectl get deploy # el default debe aparecer en blanco
    kubectl get deploy -n itsmetommy
    kubectl get pods -n itsmetommy
    ```

4. Create Service:

    ```console
    kubectl apply -f service.yaml
    kubectl get service -n itsmetommy
    ```

5. Crear HPA | Replace with the following content

    ```yaml
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: hpa-example
      namespace: itsmetommy
      labels:
            app: hpa-example
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: hpa-example
      minReplicas: 1
      maxReplicas: 10
      metrics:
      - type: Resource
        resource:
          name: cpu
          target:
            type: Utilization
            averageUtilization: 10
      - type: Resource
        resource:
          name: memory
          target:
            type: Utilization
            averageValue: 1000Mi
    ```

6. Execute

    ```console
    kubectl apply -f hpa.yaml
    kubectl get hpa -n itsmetommy
    ```

7. Activate metricserver in minikube:

    ```console
    minikube addons enable metrics-server
    minikube addons list|grep metric
    kubectl top nodes
    ```

8. Expose the service only on node1:

    ```console
    minikube service -n itsmetommy hpa-example --url
    ```

9. We write down the url that it gives us and we curl:

    ```console
    curl http://<url-anterior>
    curl -I http://<url-anterior>
    ```

10. Generate traffic and Monitor

## Abrir 4 terminales mas con ctrl+shift+t

1. In terminal n1 to see status:

    ```console
    watch kubectl -n itsmetommy get all -l app=hpa-example
    ```

2. At terminal n2 generate load:

    ```console
    service=$(minikube service -n itsmetommy hpa-example --url)
    echo $service # debe ser la misma ip que la del browser
    while true; do curl $service; done
    ```

3. In terminal n3 generate load:

    ```console
    service=$(minikube service -n itsmetommy hpa-example --url)
    while true; do curl $service; done
    ```

4. On terminal n4 generate load:

    ```console
    service=$(minikube service -n itsmetommy hpa-example --url)
    while true; do curl $service; done
    ```

5. Review hpa messages:

    ```console
    kubectl describe hpa hpa-example -n itsmetommy #Opcional: Revisar el status (watch).
    ```

6. If you wish, you can generate more load in another terminal. If a pod is pending, you can check the status using:

    ```console
    kubectl describe pod hpa-example-<pod_id> -n itsmetommy
    ```

7. Stop traffic and wait for it to begin to deescalate:

## Delete ALL resources from the cluster

```console
kubectl delete namespace itsmetommy
```
