# INGRESS

![Ingress](./img/ingress.png)

1. Activar addons de ingress

    ```console
    minikube addons enable ingress
    minikube addons list | grep ingress
    ```

2. Crear archivo web1.yaml

    ```console
    nano web1.yaml
    ```

    ```yaml
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: web1
    spec:
      selector:
        matchLabels:
          app: web1
      replicas: 2
      template:
        metadata:
          labels:
            app: web1
        spec:
          containers:
          - name: web1
            image: nginx:alpine
            ports:
              - name: http
                containerPort: 80
                protocol: TCP
    ---
    kind: Service
    apiVersion: v1
    metadata:
      name: web1-svc
    spec:
      type: NodePort
      selector:
        app: web1
      ports:
        - port: 8080
          targetPort: 80
    ```

3. Ejecutar y listar

    ```console
    kubectl apply -f web1.yaml
    kubectl get pod,deploy,svc
    ```

4. Crear archivo web2.yaml

    ```console
    nano web2.yaml
    ```

    ```yaml
    kind: Deployment
    apiVersion: apps/v1
    metadata:
     name: web2
    spec:
     selector:
       matchLabels:
         app: web2
     replicas: 2
     template:
       metadata:
         labels:
           app: web2
       spec:
         containers:
         - name: web2
           image: nginx:alpine
           ports:
             - name: http
               containerPort: 80
               protocol: TCP
    ---
    kind: Service
    apiVersion: v1
    metadata:
     name: web2-svc
    spec:
     type: NodePort
     selector:
       app: web2
     ports:
       - port: 8080
         targetPort: 80
    ```

5. Ejecutar y listar

    ```console
    kubectl apply -f web2.yaml
    kubectl get pod,deploy,svc
    ```

6. Probar los NodePorts:

    ```console
    kubectl get svc
    curl $(minikube ip):31210 # Debe salir la v1
    curl $(minikube ip):31512 # Debe salir la v2
    ```

7. Crear archivo ingress.yaml

    ```console
    nano ingress.yaml
    ```

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: web-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$1
    spec:
      rules:
      - host: hello-world.info
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web1-svc
                port:
                  number: 8080
          - path: /v2/*
            pathType: Prefix
            backend:
              service:
                name: web2-svc
                port:
                  number: 8080
    ```

8. Ejecutar y listar:

    ```console
    kubectl apply -f ingress.yaml
    kubectl get ingress
    ```

9. Probando ingress:

    ```console
    curl -H "Host: hello-world.info" $(minikube ip)
    curl -H "Host: hello-world.info" $(minikube ip)/v2
    ```

10. Simulando Domain hello-world.info

    ```console
    minikube ip
    sudo vim /etc/hosts
    ```

11. Agregamos al final la ip del minikube:

    ```vim
    192.168.49.2 hello-world.info
    ```

    ```console
    ping hello-world.info
    curl hello-world.info
    curl hello-world.info/v2
    curl hello-world.info/v2/xD
    ```

12. Limpiar todo:

    ```console
    kubectl delete all --all
    ```
