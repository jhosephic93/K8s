# LABELS

1. Crear pod_labels.yaml

```console
$ nano pod_labels.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: nginx-labels
 labels:
   release: stable
   environment: dev
   tier: backend
   region: us
spec:
 containers:
 - name: nginx
   image: nginx:alpine
```
2. Ejecutar

```console
$ kubectl apply -f pod_labels.yaml
```

3. Muestra todos los labels de un pod

```console
$ kubectl get pods --show-labels
```

4. Agregar un nuevo label a un Pod:

```console
$ kubectl label pods nginx-labels "canary=true"
$ kubectl get pods --show-labels
```

5. Actualizar labels:

```console
$ kubectl label pods nginx-labels canary=false status=ok --overwrite=true
$ kubectl get pods --show-labels
```

6. Eliminar labels “region y status” mediante edicion de pods:

```console
$ kubectl edit pods nginx-labels #Abrira un vim editor, remover la linea “region: us” y “status: ok” tecleando dos veces “d” y luego ESC, finalmente :x
$ kubectl get pods --show-labels
```


7. Mostrar labels como columnas:

```console
$ kubectl get pods -L tier,canary,environment,region
```

# LABELS SELECTORS:

1. Filtrar por release y condicional AND:

```console
$ kubectl get pods --selector="release=stable"
$ kubectl get pods --selector="release=stable,tier=backend"
$ kubectl get pods --selector="release=stable,region=us" # Este ultimo no debe devolver nada porque quitamos el label region
```

## Condicional OR

```console
$ kubectl get pods --selector="release in (stable, staging)"
$ kubectl get pods --selector="environment in (dev, prod)" -L environment
```

## Selector with Service:

1. Crear archivo selector_service.yaml con contenido:

```console
$ nano selector_service.yaml
```

```yaml
kind: Service
apiVersion: v1
metadata:
 name: nginx-selectors-svc
spec:
 type: ClusterIP
 selector:
   tier: backend
   release: stable
 ports:
 - port: 80
```

2. Ejecutar

```console
$ kubectl apply -f selector_service.yaml
$ kubectl get service
$ kubectl describe service nginx-selectors-svc
$ kubectl describe pod nginx-labels|grep IP
```

### Prestemos atencion a los valores de Labels, Selector y Endpoints | Verificar que el endpoint sea la misma ip que el pod nginx-labels.


## Actualizar el Endpoint del service automaticamente:

1. Agregar un nuevo pod_labels2.yaml con contenido

```console
$ nano pod_labels2.yaml
```
```yaml
kind: Pod
apiVersion: v1
metadata:
 name: nginx-labels2
 labels:
   release: stable
   environment: dev
   tier: backend
   region: us
spec:
 containers:
 - name: nginx2
   image: mario21ic/nginx:alpine
```

2. Ejecutar

```console
$ kubectl apply -f pod_labels2.yaml
$ kubectl describe pod nginx-labels2|grep IP
$ kubectl describe service nginx-selectors-svc
```

### Veremos que en el Service nginx-selectors-svc en Endpoints ahora existen dos ips, esto porque mediante el selector, definimos que el pod labels2 estaria dentro del Service nginx-selectors-svc

## Opcional: hacer consulta al service:

```console
$ minikube ssh
$ curl -I <ip-service>
$ exit
```
