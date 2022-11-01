# KUBERNETES | K8s

## Dato

- Kubectl funciona con Minikube/EKSCTL/K3s

## Install **Kubectl** on Linux with Binary

### Version 1.20

```console
$ wget https://dl.k8s.io/release/v1.20.0/bin/linux/amd64/kubectl
```

### Version mas reciente/stable | Cambiar "amd" por "arm"

```console
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### En cualquiera de ambos casos.

```console
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/
$ kubectl version --client
$ kubectl version --output=yaml
```

Link -> https://github.com/kubernetes/kubernetes/releases

## Ver que API-RESOURCES SOPORTA K8s Cluster.

```console
$ kubectl api-resources
```

## Revisar Status de K8s Cluster (Minkube/K3s/EKSCTL)

```bash
# Cuando usamos minikube este script es opcional / Revisar si podemos hacerlo con AWS
$ kubectl get componentstatuses
```

## Info Total de K8s

```bash
$ kubectl cluster-info
	# kuebernetes control plane = donde master se esta corriendo
	# CoreDNS = docker (ping web)
$ kubectl cluster-info dump
```

## Basic Commands.

1. Listar NODOS.

```console
$ kubectl get nodes
```

2. Listar todos los recursos:

```console
$ kubectl get all -A
```

3. Listar todos nodos y pods:

```console
$ kubectl get all
```

4. Muestra detalles de un grupo de recursos.

```console
$ kubectl describe nodes
```