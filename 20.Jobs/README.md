# JOBS

1. Crear archivo jobs.yaml con contenido:

```console
$ nano jobs.yaml
```

```yaml
kind: Job
apiVersion: batch/v1
metadata:
  name: countdown
spec:
  template:
    metadata:
      name: countdown
    spec:
      containers:
      - name: counter
        image: alpine
        command:
         - "bin/sh"
         - "-c"
         - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
      restartPolicy: Never
```

2. Crear y verificar:

```console
$ kubectl apply -f jobs.yaml
$ kubectl describe jobs countdown
$ kubectl get jobs,pods #Revisar columnas de completions, ready y status:
```

3. Revisar log del pod creado por el job:

```console
$ kubectl logs <pod-id>
```

# Jobs en Paralelo:

1. Crear archivo jobs2.yaml con contenido:

```console
$ nano jobs2.yaml
```

```yaml
kind: Job
apiVersion: batch/v1
metadata:
  name: countdown-pl
spec:
  # completions like replicas
  completions: 6
  parallelism: 2
  template:
    metadata:
      name: countdown-pl
    spec:
      containers:
      - name: counter
        image: alpine
        command:
         - "bin/sh"
         - "-c"
         - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
      restartPolicy: Never
```

```console
$ kubectl apply -f jobs2.yaml
```

2. Revisar completions y pods:

```console
$ kubectl get job,pod
```