# Security Context

- Como hacer para que los pods puedan ser ejecutados bajo ciertos usuarios y ciertos grupos de usuarios.
- Can be defined at the pod level or at the container level.
- If the container doesn’t declare its own security context, it will inherit from the parent pod.
- Generally configure two fields: the user ID that should be used to run the pod or container, and the group ID that should be used for filesystem access.

## Repaso de user:

```
$ docker run -ti -u 1000:1000 -v $PWD:/tmp alpine sh
$ echo "xD" > /tmp/hi.txt
$ exit
$ ls -la hi.txt
```

- Debe decir que el usuario es tuxito porque el UID y el GID eran los mismo que tuxito en el /etc/passwd

```console
$ grep -n "tuxit" /etc/passwd
```

1. Crear archivo pod-security-context.yaml

```console
$ nano pod-security-context.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: security-context-pod
spec:
  securityContext:
    # User ID that containers in this pod should run as
    runAsUser: 45
    # Group ID for filesystem access
    fsGroup: 231
  volumes:
    - name: simple-directory
      emptyDir: {}
  containers:
    - name: example-container
      image: alpine
      command: ["/bin/sh"]
      args: ["-c", "while true; do date >> /etc/directory/file.txt; sleep 5; done"]
      volumeMounts:
        - name: simple-directory
          mountPath: /etc/directory
```

```console
$ kubectl apply -f pod-security-context.yaml
$ kubectl get pods
```

2. Revisar security context:

```console
$ kubectl exec -ti security-context-pod -- sh
$ id
$ ps aux
$ ls -l /etc/directory/
$ tail /etc/directory/file.txt
$ date
$ exit
```

### Nota: Un usuario NO ROOT puede abrir puertos DESDE 1024 hacia arriba si quiere abrir el port 80 no podra y solo podra el usuario ROOT.

3. Opcional:

```console
$ docker run -ti --user 45:231 alpine id
$ docker run -ti --user $(id -u):$(id -g) alpine id
```

4. Crear archivo pod-sc2.yaml

```console
$ nano pod-sc2.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false # No podra escalar permisos
```

5. Va a impedir la ejecucion como root https://kubernetes.io/docs/concepts/policy/pod-security-policy/#privilege-escalation

```console
$ kubectl apply -f pod-sc2.yaml
$ kubectl exec -ti security-context-demo -- sh 
$ id
$ ps aux
$ echo "xD" > /data/demo/hi.txt
$ ls -la /data/demo/
-rw-r--r--    1 1000     2000             3 Mar 28 02:41 hi.txt
$ exit
```

6. Crear archivo pod-sc3.yaml para probar sobre escritura de context:

```console
$ nano pod-sc3.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: security-context-demo-3
spec:
 securityContext:
   runAsUser: 1000
 containers:
 - name: container-1
   image: busybox:latest
   command: [ "sh", "-c", "sleep 1h" ]
 - name: container-2
   image: busybox:latest
   command: [ "sh", "-c", "sleep 1h" ]
   securityContext:
     runAsUser: 2000
     allowPrivilegeEscalation: false
```

```console
$ kubectl apply -f pod-sc3.yaml
$ kubectl get pods
```

7. Probando los valores:

```console
$ kubectl exec security-context-demo-3 -c container-1 -- id
$ kubectl exec security-context-demo-3 -c container-2 -- id
```