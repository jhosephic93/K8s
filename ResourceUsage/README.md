# Limitar Uso de CPU & Mem

## Limitar CPU

1. Crear archivo pod-cpu.yaml con contenido:

```console
$ nano pod-cpu.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: pod-cpu-demo
spec:
 containers:
 - name: pod-cpu-demo-ctr
   #image: vish/stress
   image: michaeltinga/ubuntu22.04-stress:v1
   args:
     - --cpu
     - "2"
   resources:
     requests:
       cpu: "0.5"
     limits:
       cpu: "1"
```

2. Crear pod con comando:

```console
$ kubectl apply -f pod-cpu.yaml
$ kubectl get pods
$ kubectl describe pods/pod-cpu-demo #Inspeccionar
```

3. Revisar consumo, primero verificar que el addon de metric server este habilitado:

```console
$ minikube addons list|grep metric
$ minikube addons enable metrics-server
$ kubectl top pods pod-cpu-demo
```

4. Logs:

```console
$ kubectl logs -f pods/pod-cpu-demo
$ sudo htop  #Revisar consumo de cpu:
```

## Seteando CPU 5.

- Crear pod_cpu2.yaml con contenido:

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: pod-cpu-demo-max
spec:
 containers:
 - name: pod-cpu-demo-ctr
   #image: vish/stress
   image: michaeltinga/ubuntu22.04-stress:v1
   args:
     - -cpus
     - "5"
   resources:
     limits:
       cpu: "5"
```

6. Aplicar:

```console
$ kubectl apply -f pod_cpu2.yaml
$ kubectl get pods
```

7. Inspeccionar -  Veremos que pasa a pending | describe veremos que dice no hay disponibles.

```console
$ kubectl describe pods pod-cpu-demo-max
```

8. Revisar consumo en nodo em Limits dice 1(25%) por el pod-cpu-demo

```console
$ kubectl describe node
```

9. Editar pod_cpu2.yaml para dejar solo con 2 unidades de cpu como sigue:
```yaml
       cpu: "2"
```

```console
$ kubectl delete -f pod_cpu2.yaml
$ kubectl apply -f pod_cpu2.yaml
$ kubectl top nodes #Verificar consumo en nodo:
$ kubectl describe nodes
```

10. Verificar que el **REQUEST DE CPU ES EL MISMO Que EL LIMIT**, porque no se definio:

```console
$ kubectl describe pods pod-cpu-demo-max
```

## Exceder el uso de Cpu:

```console
$ kubectl delete pods --all
$ kubectl describe nodes
```

- Crear archivo con pod_cpu3.yaml contenido:

```console
$ nano pod_cpu3.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: pod-cpu-max
spec:
  containers:
  - name: pod-cpu-max
    image: ubuntu:18.04
    command: ["/bin/sh"]
    args: ["-c", "while true; do date; sleep 5; done"]
    resources:
      limits:
        cpu: "2"
```

```console
$ kubectl apply -f pod_cpu3.yaml
```

1. Verificar los recursos usados en cpu del nodo:

```console
$ kubectl describe nodes
```

2. Debe salir mas o menos asi:

| Resource  | Requests    | Limits  |
| :-------- |:-----------:| -------:|
| cpu       | 2750m (68%) | 2 (50%) |

3. Instalar stress e intentar usar las cpus asignadas:

```console
$ kubectl exec -ti pod-cpu-max -- bash
# apt update
# apt install -y stress
# stress --cpu 2
```

4. Luego cambiar a --cpu 4 

```console
# stress --cpu 4
```

5. En otro terminal verificar uso de cpu:

```console
$ sudo htop
$ kubectl top pods
```
## RESOURCE MEMORY | Limitar MEM

1. Crear archivo pod_mem.yaml con contenido:

```console
$ nano pod_mem.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: pod-memory-demo
spec:
 containers:
 - name: memory-demo-ctr
   #image: polinux/stress
   image: michaeltinga/ubuntu22.04-stress:v1
   resources:
     limits:
       memory: "200Mi"
     requests:
       memory: "100Mi"
   command: ["stress"]
   args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

2. Ejecutar

```console
$ kubectl apply -f pod_mem.yaml
$ kubectl get pods
$ kubectl describe pods pod-memory-demo
```

3. Verificar uso de memoria:

```console
$ kubectl top pods
$ kubectl top nodes
$ kubectl describe nodes
```

4. Inspeccionar:

```console
$ kubectl exec -ti pod-memory-demo -- sh
# free -m
# ps aux
# exit
```

## Exceder Memory:

1. Crear archivo pod_mem2.yaml con:

```console
$ nano pod_mem2.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: pod-memory-demo-max
spec:
 containers:
 - name: memory-demo-ctr
   #image: polinux/stress
   image: michaeltinga/ubuntu22.04-stress:v1
   resources:
     limits:
       memory: "200Mi"
   command: ["stress"]
   args: ["--vm", "1", "--vm-bytes", "350M", "--vm-hang", "1"]
```

2. Crear aplicando:

```console
$ kubectl apply -f pod_mem2.yaml
$ kubectl describe pods pod-memory-demo-max
```

3. Revisar consumo y comportamiento:

```console
$ watch kubectl top pods
```