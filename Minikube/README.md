# MINIKUBE

## Pre-requisites

- Install Kubectl

## Install **Minikube** on Linux with Binary

### Version 1.17

```console
wget https://github.com/kubernetes/minikube/releases/download/v1.17.1/minikube-linux-amd64
```

### Version mas reciente/stable | Cambiar "amd" por "arm"

```console
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

### En cualquiera de ambos casos

```console
sudo chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

### Iniciar Minikube

```console
minikube start
minikube status
kubectl get nodes
```

Link -> <https://github.com/kubernetes/minikube/releases>

## Minikube Dashboard

- Primera terminal.

    ```console
    minikube dashboard --url=true
    ```

- Segunda terminal.

    ```console
    kubectl proxy --address='0.0.0.0' --disable-filter=true
    ```

- Ingresar en el navegador.

    ```html
    http://192.168.1.50:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
    ```

## Entrar en minikube por ssh

```console
$ minikube ssh
> docker ps -a
> exit
```

## PLUGINS DE MINIKUBE

### Listar Plugins de Minikube

```console
minikube addons list
```

### Activar plugins Metrics de Minikube

```console
minikube addons enable metrics-server
minikube addons list
```

### Verificar nodos

```bash
minikube addons list
# Ver métricas de todos los nodes, pods.
kubectl top nodes
kubectl top pods
```
