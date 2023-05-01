# ANNOTATIOONS:

1. Crear archivo pods_annotations.yaml

```console
$ nano pods_annotations.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
 name: nginx-annotations
 annotations:
   commit: 44e3275e0
   logs: 'http://logservice.com/nginx'
   contact: 'Mario Inga <mario21ic@gmail.com>'
spec:
 containers:
 - name: nginx
   image: nginx:alpine
```

2. Ejecutar.

```console
$ kubectl apply -f pods_annotation.yaml
$ kubectl describe pods nginx-annotations
```
