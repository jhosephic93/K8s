# K8s cluster on Aws:

1. Instalar aws cli v2:

```console
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
$ aws --version
```

2. Conectar aws cli con cuenta | configurar un Access Key & Secret Key:

```console
$ aws configure --profile test
$ aws ec2 describe-instances
```

# Instalar eksctl:

```console
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
$ sudo chmod +x /usr/local/bin/eksctl
$ eksctl version
```

# Crear o actualizar kubeconfig:

```console
$ rm ~/.kube/config
$ aws eks --region region update-kubeconfig --name cluster_name
$ kubectl get pods --kubeconfig ./.kube/config
$ kubectl get svc
```

## Ejemplo de crear archivo cluster.yaml

```console
$ nano cluster.yaml
```

```yaml
kind: ClusterConfig
apiVersion: eksctl.io/v1alpha5
metadata:
  name: demo1
  region: us-east-1
nodeGroups:
  - name: ng-1
    instanceType: t3.small
    desiredCapacity: 2
    availabilityZones: ["us-east-1a", "us-east-1b"]
```

```console
$ eksctl create cluster -f cluster.yaml
```

1. Probar conexion y despliegue

```console
$ kubectl get nodes
$ kubectl scale deploy nginx-dp --replicas=5 #Escalar:
$ eksctl get clusters --region=us-west-2 #Listar clusters:
$ eksctl delete cluster demo2 --region=us-west-2 #Eliminar cluster:
```
