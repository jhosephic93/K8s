# CronJob

1. Crear archivo cronjob.yaml con contenido

```console
$ nano cronjob.yaml
```

```yaml
kind: CronJob
apiVersion: batch/v1
metadata:
  name: cj-countdown
spec:
  # minuto, hora, dia del mes, dia de la semana y mes
  schedule: "2 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cj-counter
            image: alpine
            command:
            - "bin/sh"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
          restartPolicy: OnFailure
```

2. Ejecutar

```console
$ kubectl apply -f cronjob.yaml
$ kubectl get cronjob
```

3.  Averiguar la hora del master de minikube:

```console
$ minikube ssh
# date
# exit
```

## CronJob para cada minuto:

1. Crear archivo cronjob2.yaml

```console
$ nano cronjob2.yaml
```

```yaml
kind: CronJob
apiVersion: batch/v1
metadata:
  name: cj-countdown2
spec:
  schedule: "*/1 * * * *"
  successfulJobsHistoryLimit: 2
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cj-counter
            image: alpine
            command:
            - "bin/sh"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
          restartPolicy: OnFailure
```

2. Ejecutar

```console
$ kubectl apply -f cronjob2.yaml
$ kubectl get cronjob
$ watch kubectl get cronjob,job,pods
```

3. Eliminar todos los cronjobs, jobs y demas:

```console
$ kubectl delete cronjob,jobs,all --all
```