# Readiness

## To test if the pod is healthy and Ready

1. Create file deploy_readiness_liveness.yaml

    ```console
    nano deploy_readiness_liveness.yaml
    ```

    ```yaml
    kind: Deployment
    apiVersion: apps/v1
    metadata:
     name: nginx-dp-liveness
    spec:
     selector:
       matchLabels:
         app: nginx
     replicas: 2
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:alpine
           readinessProbe:
             httpGet:
               path: /ready
               port: 80
             periodSeconds: 2
             initialDelaySeconds: 0
             failureThreshold: 3
             successThreshold: 1
           ports:
             - name: web
               containerPort: 80
               protocol: TCP
    ```

2. Apply

    ```console
    kubectl apply -f deploy_readiness_liveness.yaml
    kubectl get pods # Column “status” should be in “running” but the “ready” column should be in “0/1”
    ```

    ```console
    kubectl describe pod <pod-id> # Something like this should appear: Readiness probe failed: HTTP probe failed with statuscode: 404
    ```

3. Test path /ready

    ```console
    kubectl exec -ti nginx-dp-liveness-8cf79d5c7-r9rff -- sh
    apk add curl
    curl localhost/ready #An error appears because the route does not exist
    curl localhost/
    exit
    ```

## Try with Service

1. To make it work, we create a service_readiness_liveness.yaml

    ```console
    nano service_readiness_liveness.yaml
    ```

    ```yaml
    kind: Service
    apiVersion: v1
    metadata:
     name: nginx-np-readiness-liveness
    spec:
     type: NodePort
     selector:
       app: nginx
     ports:
       - name: web
         port: 80 # service interno
         targetPort: 80 # container
    ```

2. Execute

    ```console
    kubectl apply -f service_readiness_liveness.yaml
    kubectl get svc
    kubectl describe service/nginx-np-readiness-liveness #The Endpoints section must be blank
    curl $(minikube ip):32002 # Sale conexion refused
    ```

3. Edit file deploy_readiness_liveness.yaml leaving the line as follows:
    path: /

    ```console
    nano deploy_readiness_liveness.yaml
    ```

4. Execute again:

    ```bash
    kubectl apply -f deploy_readiness_liveness.yaml
    kubectl get pods
    kubectl describe service/nginx-np-readiness-liveness
    ```

5. Now you should find the 2 healthy pods similar to:

    ```vim
    Endpoints:                172.17.0.4:80,172.17.0.6:80
    ```

**********************************

## Liveness

1. To restart a pod. Modify file deploy_readiness_liveness.yaml

    ```console
    nano deploy_readiness_liveness.yaml
    ```

    ```yaml
    kind: Deployment
    apiVersion: apps/v1
    metadata:
     name: nginx-dp-liveness
    spec:
     selector:
       matchLabels:
         app: nginx
     replicas: 2
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: mario21ic/nginx:alpinev1
           readinessProbe:
             httpGet:
               path: /
               port: 80
             initialDelaySeconds: 0
             periodSeconds: 2
             failureThreshold: 3
             successThreshold: 1
           livenessProbe:
             httpGet:
               path: /live
               port: 80
             initialDelaySeconds: 5
             timeoutSeconds: 1
             periodSeconds: 10
             failureThreshold: 3
           ports:
             - name: web
               containerPort: 80
               protocol: TCP
    ```

2. Execute

    ```console
    kubectl apply -f deploy_readiness_liveness.yaml
    kubectl get deploy
    kubectl get pods #It should be running, but it has several restarts.
    ```

3. To try entering one of the pods:

    ```console
    kubectl exec -ti <pod-id> -- sh  #Pod sera terminado por liveness
    apk add curl
    curl localhost/live
    ```

4. Describe

    ```console
    kubectl describe pod #Debe salir error:
    ```

5. Liveness probe failed: HTTP probe failed with statuscode: 404

    ```console
    kubectl describe service/nginx-np-readiness-liveness
    ```

6. Fix the file deploy_readiness_liveness.yaml
    path: /

    ```console
    nano deploy_readiness_liveness.yaml
    ```

7. Execute again

    ```console
    kubectl apply -f deploy_liveness.yaml
    kubectl get pods
    ```

8. Should be deleting the previous two and generating 2 new pods and testing again.

    ```console
    kubectl describe svc nginx-np-readiness-liveness
    curl $(minikube service nginx-np-readiness-liveness --url)
    ```
