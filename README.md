# KUBERNETES | K8s

## Tip

- Kubectl funciona con Minikube/EKSCTL/K3s

## Install **Kubectl** on Linux with Binary

### Version 1.20

```console
wget https://dl.k8s.io/release/v1.20.0/bin/linux/amd64/kubectl
```

### Most recent/stable version | Change "amd" by "arm"

```console
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### In either case

```console
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
kubectl version --output=yaml
```

Link -> <https://github.com/kubernetes/kubernetes/releases>

## See which API-RESOURCES SUPPORTS K8s Cluster

```console
kubectl api-resources
```

## Check Status of K8s Cluster (Minkube/K3s/EKSCTL)

```bash
# Cuando usamos minikube este script es opcional / Revisar si podemos hacerlo con AWS
$ kubectl get componentstatuses
```

## Total K8s Info

```bash
$ kubectl cluster-info
# kuebernetes control plane = donde master se esta corriendo
# CoreDNS = docker (ping web)
$ kubectl cluster-info dump
```

## Basic Commands

1. List NODES.

  ```bash
  kubectl get nodes
  ```

2. Listar todos los recursos:

  ```console
  kubectl get all -A
  ```

3. Listar todos nodos y pods:

  ```console
  kubectl get all
  ```

4. Muestra detalles de un grupo de recursos.

  ```console
  kubectl describe nodes
  ```
