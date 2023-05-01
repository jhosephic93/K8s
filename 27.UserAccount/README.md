# User Account:

- When you, a human, make a request to Kubernetes’ API Server using kubectl, you are authenticated through a User Account.

```console
$ kubectl config current-context # Debe mostrarnos minikube como valor unico.
```

1. Creando certificados para user1

```console
$ openssl genrsa -out user1.key 2048
$ openssl req -new -key user1.key -out user1.csr -subj "/CN=user1/O=group1"
$ ls -la user1.*
$ ls -la /home/$USER/.minikube/ca.*
$ openssl x509 -req -in user1.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out user1.crt -days 500
$ ls -la user1.*
```

2. Crear user1:

```console
$ kubectl config set-credentials user1 --client-certificate=user1.crt --client-key=user1.key
$ kubectl config set-context user1-context --cluster=minikube --user=user1
```

3. Revisar la config:

```console
$ kubectl config view
$ ls -la ~/.kube/config
```

4. Cambiar de context:

```console
$ kubectl config use-context user1-context
$ kubectl config current-context
$ kubectl get pods #Debe salir error de permisos porque aun no tiene roles asignados.
```

5. Creando RBAC:

```console
$ kubectl config use-context minikube
```

6. Crear archivo ua-role.yaml

```console
$ nano ua-role.yaml
```

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```console
$ kubectl apply -f ua-role.yaml
```

7. Crear archivo ua-rolebinding.yaml


```console
$ nano ua-rolebinding.yaml
```

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # must match the name of the Role
  apiGroup: rbac.authorization.k8s.io
```

```console
$ kubectl apply -f ua-rolebinding.yaml
$ kubectl get role,rolebinding
```

8. Probando permisos:

```console
$ kubectl config use-context user1-context
$ kubectl get pods
$ kubectl get pods -w
```

9. Ahora si debe permitir listar los pods

```console
$ kubectl get deploy #Debe dar error porque no tiene permisos para listar deployments. #Error from server (Forbidden): deployments.apps is forbidden: User "user1" cannot list resource "deployments" in API group "apps" in the namespace "default"
```

### Nota: Por si sale error en minikube, eliminar y volver a crear:

10. Dato: Este procedimiento elimina todos los recuros/objetos dentro de k8s

```console
$ minikube delete
$ rm -rf ~/.minikube
$ minikube start
$ minikube status
```

### Dinamica:

    - Puedan eliminar los pods $ kubectl delete pods --all
        - Solucion: cambiar al context minikube y agregar “delete” en verbs del ua-role.yaml.
    - Darle permisos al user1 de listar deployments
    - Darle permisos al user1 de poder ejecutar $ kubectl get services

11. Archivo ua-role2.yaml

```console
$ nano ua-role2.yaml
```

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: deployment-reader
rules:
- apiGroups: ["apps"] # "" indicates the core API group
  resources: ["deployments"]
  verbs: ["get", "watch", "list"]
```


12. Archivo ua-rolebinding2.yaml

```console
$ nano ua-rolebinding2.yaml
```

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-deployments
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: deployment-reader # must match the name of the Role
  apiGroup: rbac.authorization.k8s.io
```

```console
$ kubectl config use-context minikube
$ kubectl apply -f ua-role2.yaml
$ kubectl apply -f ua-rolebinding2.yaml
```

13. Probar:

```console
$ kubectl config use-context user1-context
$ kubectl get deploy
```

*************

## Opcional: Instalar kubeselect

```console
$ wget https://github.com/fatliverfreddy/kubeselect/raw/master/kubeselect
$ chmod +x kubeselect
$ sudo mv kubeselect /usr/local/bin/
$ kubeselect #Seleccionar el user-context a usar ya sea minikube o user1-context
```

#### Nota: tambien pueden probar kubie https://github.com/sbstp/kubie y aqui tutorial de uso https://www.youtube.com/watch?v=kiRTQVUzkh8 