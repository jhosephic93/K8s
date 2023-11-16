# CronJob

1. Create file cronjob.yaml

    ```console
    nano cronjob.yaml
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

2. Execute

    ```console
    kubectl apply -f cronjob.yaml
    kubectl get cronjob
    ```

3. Find out the time of the minikube master:

    ```console
    minikube ssh
    date
    exit
    ```

## CronJob for every minute

1. Create file cronjob2.yaml

    ```console
    nano cronjob2.yaml
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

2. Execute

    ```console
    kubectl apply -f cronjob2.yaml
    kubectl get cronjob
    watch kubectl get cronjob,job,pods
    ```

3. Delete all cronjobs, jobs and more:

    ```console
    kubectl delete cronjob,jobs,all --all
    ```
