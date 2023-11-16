# Instalando K8s Onpromise con K3S - Binario

## Pre-requisitos

- En AMBOS NODOS descargar k3s.
- En ambos nodos debe estar instalado docker.

1. Version 1.18

    ```console
    wget https://github.com/k3s-io/k3s/releases/download/v1.18.17%2Bk3s1/k3s
    ```

1. DEMO Michaelt ARM64

    ```console
    wget https://github.com/k3s-io/k3s/releases/download/v1.25.2%2Bk3s1/k3s-arm64
    chmod +x k3s
    sudo mv k3s /usr/local/bin/
    k3s --version
    ```

## En Nodo1

```console
sudo k3s server # En caso salga error revisar “sudo lsmod | grep vxlan”
```

## En otro terminal en Nodo1

```console
sudo k3s kubectl get nodes
sudo ls -la /etc/rancher/k3s/k3s.yaml
sudo cat /var/lib/rancher/k3s/server/node-token  #Anotamos el token
```

## En Nodo2

```console
sudo k3s agent --server https://<ip-lan-nodo1>:6443 --token <token-anterior> 
```

## En otro terminal en Nodo1

```console
sudo k3s kubectl get nodes
sudo k3s kubectl apply -f deployment/deploy.yaml
sudo k3s kubectl get pods,deploy #Dato: Si en el nodo1 donde corri el script sudo k3s server presiono ctrl + c, se desconectara de k3s pero los pods y deploys creados seguiran ejecutandose.
```

### Mas info <https://k3s.io/>

******

## Instalar K3s Forma automatizada con K8s

## En Nodo 1

```console
curl -sfL https://get.k3s.io | sh -
sudo service k3s status
sudo k3s kubectl get nodes
sudo cat /var/lib/rancher/k3s/server/node-token 
```

## En Nodo2

```console
export K3S_URL=https://<ip-lan-nodo1>:6443
export K3s_TOKEN=<token-anterior>
curl -sfL https://get.k3s.io | sh -
sudo service k3s-agent status
```

## En Nodo1

```console
sudo k3s kubectl get nodes #Ya debe aparecer el nodo2 como parte del k8s cluster
sudo k3s kubectl get pods,deploy
```

## Desinstalar K8s cluster

- Nodo2:

    ```console
    sudo /usr/local/bin/k3s-agent-uninstall.sh
    ```

- Nodo1:

    ```console
    sudo /usr/local/bin/k3s-uninstall.sh
    ```

- Ejemplo de instalar K8s con K3s en produccion <https://gitlab.com/k3s_hetzner/k3s_hetzner>
