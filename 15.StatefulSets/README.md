# StatefulSets

- DATO: Esta demo por la imagen solo funciona en amd64.
- No crea ReplicaSet, genera Pods directamente.
- Cada réplica del pod tendrá su propio volumen, y cada uno creará su propio PVC (PersistentVolumeClaim).
- Escalan en orden, primero POD-1, luego POD-2 y asi sucesivamente.
- Pod’s Ordinal Index (Escala de 0 a + y desescala de + a 0)
- Cada POD tiene una especia de servicio por debajo, tienen nombre con el cual se puede llegar a cada pod, para que entre ellos se pueda realizar consultas.

![](./img/statefulsets.jpg)

# Stateful vs Stateless

- Stateless app:
    - Depends on no persistent storage.
    - Apps with a single function or service, examples: web, print, cdn, etc.
- Stateful app:
    - Requires persistent storage.
    - Has several other parameters which it is supposed to look after in cluster.
    - Examples: databases, transaction solutions, such as home banking, mail servers.

1. Revisando persistent volumes:

```console
$ kubectl get pv
```

2. Crear archivo statefulset.yaml con contenido:

```
$ nano statefulset.yaml
```

```yaml
kind: Service
apiVersion: v1
metadata:
 name: nginx
 labels:
   app: nginx
spec:
 ports:
 - port: 80
   name: web
 clusterIP: None
 selector:
   app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
 name: web
spec:
 serviceName: "nginx"
 replicas: 2
 selector:
   matchLabels:
     app: nginx
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: k8s.gcr.io/nginx-slim:0.8
       ports:
       - containerPort: 80
         name: web
       volumeMounts:
       - name: www
         mountPath: /usr/share/nginx/html
 volumeClaimTemplates:
 - metadata:
     name: www
   spec:
     accessModes: [ "ReadWriteOnce" ]
     resources:
       requests:
         storage: 10Mi
```

2. Ejecutar

```console
$ kubectl apply -f statefulset.yaml
$ kubectl get statefulset
$ kubectl get sts
$ kubectl get pods,pvc,pv
```

3. Verificar si el service lo encuentra:

```console
$ kubectl get svc
$ kubectl describe svc nginx
```

4. Probando Pods names:

```console
$ kubectl delete pods web-1
$ kubectl get pods
$ kubectl delete pods web-0
$ kubectl get pods
```

5. Probando hostname:

```console
$ for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
```

6. Probando DNS:

```console
$ kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm /bin/sh
# nslookup web-0.nginx
# nslookup web-1.nginx
# exit
$ kubectl describe pod web-1 |grep IP
$ kubectl describe pod web-0 |grep IP
```

7. Probando storage:

```console
$ for i in 0 1; do kubectl exec web-$i -- sh -c 'echo $(hostname) > /usr/share/nginx/html/index.html'; done
$ for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
$ kubectl delete pod -l app=nginx
```

8. En otro terminal ver los cambios:

```console
$ watch kubectl get pods #Observar que no se generan nuevos pods hasta que se terminen porque son de nombres unicos.
```

9. Pruebo nuevamente si los cambios persisten:

```console
$ for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
$ kubectl exec -ti web-0 -- bash
# cat /usr/share/nginx/html/index.html
# hostname
# exit
```

10. Probando sts scaling:

```console
$ kubectl scale sts web --replicas=5
$ kubectl get sts
$ kubectl get pods
$ for i in 0 1 2 3 4; do kubectl exec -it web-$i -- curl localhost; done #Debe salir 404 para los 3 nuevos pods.
```

11. Inspeccionando errores:

```console
$ kubectl describe pods web-2
$ kubectl get pvc
$ kubectl describe pvc www-web-2
$ kubectl get pv #Revisando pv y pvc:
$ kubectl get pvc
$ kubectl scale sts web --replicas=2 #Down scaling:
$ kubectl get pv #Revisando pv y pvc:
$ kubectl get pvc
```

12. Tip: Recuperar manifest de un pod:

```console
$ kubectl get pod web-2 -o yaml
```
