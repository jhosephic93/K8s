# RKE con Nodo1 y Nodo2:

## RKE:

```
$ wget https://github.com/rancher/rke/releases/download/v1.2.7/rke_linux-amd64 
$ chmod +x rke_linux-amd64
$ sudo mv rke_linux-amd64 /usr/local/bin/rke
$ rke --version
```

## En Nodo 1:

1. Clonar repositorio.

```console
$ git clone https://github.com/mario21ic/k8s-demos
$ cat k8s-demos/rke/rancher-cluster.yml
```

```console
$ ls -la ~/.ssh/id_rsa
$ ssh-keygen   #Dar enter a todo, solo en caso de que no tengan el archivo ~/.ssh/id_rsa
```

- Editar archivo rancher-cluster.yml
    - En la linea 3 dejar el valor de IP Lan Nodo1
    - En la linea 12 dejar el valor de IP Lan Nodo2
    - Reemplazar la linea 6 y 15 cambiando vagrant por tuxito

## En Nodo1 copiar ssh-keys para cada nodo:

```console
$ ssh-copy-id tuxito@ip-lan-nodo1
$ ssh-copy-id tuxito@ip-lan-nodo2 #La clave es Dockercito.123
```

### K3s vs rke minuto 04:15:00 Video 07

## Probar conexion sin clave:

```console
$ ssh tuxito@ip-lan-nodo1 id
$ ssh tuxito@ip-lan-nodo2 id
```

## Levantar componentes Nodo1:

```console
$ rke up --config rancher-cluster.yml 
$ export KUBECONFIG=kube_config_rancher-cluster.yml
$ kubectl get nodes
```

## Probando desplegar pods:

```console
$ kubectl apply -f ../deployment/deploy.yaml
$ kubectl get pods
$ kubectl apply -f ../configmap/configmap.yaml
$ kubectl get pods,deploy,cm
```

******

# Install VAGRANT

1. Dependencias VAGRANT y VIRTUALBOX | No tiene que ver con rke:

```console
$ sudo apt install virtualbox unzip -y
$ wget https://releases.hashicorp.com/vagrant/2.2.7/vagrant_2.2.7_linux_amd64.zip 
$ unzip vagrant_2.2.7_linux_amd64.zip 
$ sudo mv vagrant /usr/local/bin/
$ vagrant --version
```
