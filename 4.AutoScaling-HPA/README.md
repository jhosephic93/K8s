# Autoscaling de Pods | HPA - Horizontal Pods AutoScaling

- Two types
  - Vertical = Modificar el tamaño de un EC2
  - Horizontal
    - Horizontal pod Scaling -> $ kubectl scale deploy nginx-dp --replicas=3
- K8s permite definir una política sencilla para realizar HPA (Horizontal Scaling Pods) automatico.
- La métrica usada para esto es el consumo de CPU, se permiten métricas personalizadas, pero cada instalación es responsable de generarlas, pues aún no hay un estandar definido.
- Heapster/Metricserver debe estar activado para que funcione correctamente.
- Revisar la configuración del controller-manager para detalles de la metrica.

1. Clonar repositorio:

```console
$ git clone https://github.com/mario21ic/kubernetes-hpa-example-minikube.git
$ cd kubernetes-hpa-example-minikube
```

2. Crear namespace:

```console
$ kubectl create namespace itsmetommy
$ kubectl get namespace
```

3. Crear deployment:

```console
$ kubectl apply -f deployment.yaml
$ kubectl get deploy # el default debe aparecer en blanco
$ kubectl get deploy -n itsmetommy
$ kubectl get pods -n itsmetommy
```
4. Crear service:

```console
$ kubectl apply -f service.yaml
$ kubectl get service -n itsmetommy
```

5. Crear HPA | Remplazar con el siguiente contenido

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-example
  namespace: itsmetommy
  labels:
        app: hpa-example
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-example
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 10
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageValue: 1000Mi
```

5. Ejecutar

```console
$ kubectl apply -f hpa.yaml
$ kubectl get hpa -n itsmetommy
```

6. Activar metricserver en minikube:

```console
$ minikube addons enable metrics-server
$ minikube addons list|grep metric
$ kubectl top nodes
```

7. Exponer el service solo en el nodo1:

```console
$ minikube service -n itsmetommy hpa-example --url
```

8. Anotamos la url que nos arroja y hacemos curl:

```console
$ curl http://<url-anterior>
$ curl -I http://<url-anterior>
```

9. Generar trafico y Monitorear

## Abrir 4 terminales mas con ctrl+shift+t

1. En el terminal n1 para ver status:

```console
$ watch kubectl -n itsmetommy get all -l app=hpa-example
```

2. En el terminal n2 generar carga:

```console
$ service=$(minikube service -n itsmetommy hpa-example --url)
$ echo $service # debe ser la misma ip que la del browser
$ while true; do curl $service; done
```

3. En el terminal n3 generar carga:

```console
$ service=$(minikube service -n itsmetommy hpa-example --url)
$ while true; do curl $service; done
```

4. En el terminal n4 generar carga:

```console
$ service=$(minikube service -n itsmetommy hpa-example --url)
$ while true; do curl $service; done
```

5. Revisar los mensajes del hpa:

```console
$ kubectl describe hpa hpa-example -n itsmetommy #Opcional: Revisar el status (watch).
```

6. Si desean pueden generar mas carga en otro terminal. Si un pod esta en pending pueden revisar el status mediante:

```console
$ kubectl describe pod hpa-example-<pod_id> -n itsmetommy
```

7. Parar el trafico y esperar que empiece a desescalar:

## Eliminar TODOS los recursos del cluster:

```console
$ kubectl delete namespace itsmetommy
```
