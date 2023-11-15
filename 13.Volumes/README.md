# K8s VOLUMES

- Volume Types
  - EMPTYDIR
    - Emptydir is a volume on the fly, it has no place, it is saved in memory, it is ephemeral.
  - NFS
    - The example was not done because an NFS Server is needed.
  - HostPath
    - It is created in a specific path of the Kubernetes Cluster node.
  - CONFIGMAP
    - Stores configuration values (key-value pairs)
    - Values consumed in pods such as: (1. Environment variables 2.Files)
    - Helps separate code from configuration.
  - SECRET
    - To store and manage sensitive data (passwords, tokens, keys)
    - Referenced as files in a volume, mounted from a secret.
    - Base64 encoded.
    - Types: generic, Docker registry, TLS

**************************************

## HostPath

- Persistent Volume
- Persistent Volume Claim:
- Pod use Claim:

## Persistent Volume

1. Create file pv.yaml with content:

    ```console
    vim pv.yaml
    ```

    ```yaml
    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: my-pv
      labels:
        type: local
    spec:
      # Match con el pvc
      storageClassName: manual
      capacity:
        storage: 100Mi
      accessModes:
        - ReadWriteOnce
      hostPath:
        # Se ejecuta en nodo y debe tener permisos
        path: "/tmp/datapv"
    ```

2. Create and check:

    ```console
    kubectl apply -f pv.yaml 
    kubectl get persistentvolume
    kubectl get pv
    kubectl describe persistentvolume my-pv
    ```

### Persistent Volume Claim

1. Create file pvc.yaml with content:

    ```console
    vim pvc.yaml
    ```

    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: myclaim
    spec:
      # Match con el pv
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 8Mi
    ```

2. Create and check:

    ```console
    kubectl apply -f pvc.yaml 
    kubectl get persistentvolumeclaim
    kubectl get pvc
    kubectl describe persistentvolumeclaim/myclaim
    ```

### Pod using the Claim

1. Create file pod_pv_pvc.yaml:

    ```console
    vim pod_pv_pvc.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: alpine
    spec:
      volumes:
        - name: mi-volume
          persistentVolumeClaim:
            claimName: myclaim
      containers:
        - image: alpine
          name: alpine
          command:
            - sleep
            - "3600"
          volumeMounts:
          - mountPath: /mnt/mydata
            name: mi-volume
    ```

2. Create and check:

    ```console
    kubectl apply -f pod_pv_pvc.yaml
    kubectl get pods
    kubectl exec -ti alpine sh
    mkdir /mnt/mydata/hello
    echo "xD" > /mnt/mydata/hello/hi.txt
    exit
    ```

3. Delete pod and test persistence:

    ```console
    kubectl delete -f pod_pv_pvc.yaml
    kubectl apply -f pod_pv_pvc.yaml
    kubectl exec -ti alpine sh
    cat /mnt/mydata/hello/hi.txt
    exit
    ```

4. Verify the hostPath on the K8s cluster:

    ```console
    minikube ssh
    ls -la /tmp/datapv/hello/
    cat /tmp/datapv/hello/hi.txt
    exit
    ```

**************************************

## Volumen emptyDir

1. Create file pod_emptydir.yaml

    ```console
    nano pod_emptydir.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: alpine-empty
    spec:
      volumes:
        - name: scratch-volume
          emptyDir: {}
      containers:
        - image: alpine
          name: alpine
          command:
            - sleep
            - "3600"
          volumeMounts:
          - mountPath: /scratch
            name: scratch-volume
    ```

2. Create and check:

    ```console
    kubectl apply -f pod_emptydir.yaml
    kubectl get pod
    kubectl exec -ti alpine-empty sh
    echo "xD" > /scratch/hello.txt
    exit
    ```

### Fact: since a volume is emptydir on the fly it has no place, it is saved in memory

1. Delete Pod and test persistence:

    ```console
    kubectl delete -f pod_emptydir.yaml
    kubectl apply -f pod_emptydir.yaml
    kubectl exec -ti alpine-empty sh
    ls -la /scratch/
    ```

**************************************

## ConfigMaps

### Configmaps

1. Command Basics

    ```console
    kubectl edit configmap aws-auth -n kube-system
    kubectl get configmap -n kube-system
    kubectl get configmap aws-auth -n kube-system
    kubectl describe configmap aws-auth -n kube-system
    ```

1. Create file configmap1.yaml:

    ```console
    vim configmap1.yaml
    ```

    ```yaml
    kind: ConfigMap
    apiVersion: v1
    metadata:
     name: nginx-cm
    data:
     MY_KEY: "MY_VALUE"
     foo: bar
     special.how: very
     special.type: charm
    ```

    ```console
    kubectl apply -f configmap1.yaml
    kubectl get configmap
    kubectl describe configmap nginx-cm
    ```

### Pod using ConfigMap

1. Create file pod_cm.yaml:

    ```console
    nano pod_cm.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
     name: nginx-pod-cm
    spec:
     containers:
     - name: nginx
       image: nginx:alpine
       env:
         - name: MY_KEY
           value: "MY_VALUE2"
         - name: foo
           valueFrom:
             configMapKeyRef:
               name: nginx-cm
               key: foo
         - name: SPECIAL_LEVEL_KEY
           valueFrom:
             configMapKeyRef:
               name: nginx-cm
               key: special.how
    ```

2. Execute

    ```console
    kubectl apply -f pod_cm.yaml 
    kubectl get pods
    kubectl exec -ti nginx-pod-cm sh
    echo $foo
    echo $SPECIAL_LEVEL_KEY
    env
    exit
     ```

3. Create file pod_cm_all.yaml

    ```console
    nano pod_cm_all.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-pod-cm-all
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        envFrom:
          - configMapRef:
              name: nginx-cm
    ```

4. Execute

    ```console
    kubectl apply -f pod_cm_all.yaml
    kubectl get pods
    kubectl exec -ti nginx-pod-cm-all sh
    ```

**************************************

## ConfigMap as File

1. Create file configmap2.yaml

    ```console
    vim configmap2.yaml
    ```

    ```yaml
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: myconfigdata
    data:
      myfile.txt: |
        This is a file demo
        this is a second line

        this is a ...
    ```

2. Execute

    ```console
    kubectl apply -f configmap2.yaml
    kubectl describe cm myconfigdata
    ```

### Pod using ConfigMap as file

1. Create file pod_cm_file.yaml:

    ```console
    nano pod_cm_file.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-pod-cm-file
    spec:
     volumes:
       - name: myfile
         configMap:
           name: myconfigdata
     containers:
       - name: nginx
         image: nginx:alpine
         volumeMounts:
         - name: myfile
           mountPath: /mnt/files/
    ```

    ```console
    kubectl apply -f pod_cm_file.yaml
    kubectl get pods
    kubectl exec -ti nginx-pod-cm-file sh
    ls /mnt/files/
    cat /mnt/files/myfile.txt
    exit
    ```

### Optional

1. Same implementation, but with Deployment deploy_cm.yaml

    ```console
    nano deploy_cm.yaml
    ```

    ```yaml
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: nginx-dep
      labels:
        app: nginx
    spec:
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          volumes:
            - name: myfile
              configMap:
                name: myconfigdata
          containers:
            - name: nginx
              image: nginx:alpine
              ports:
                - containerPort: 80
              volumeMounts:
                - name: myfile
                  mountPath: /mnt/files/
    ```

    ```console
    kubectl apply -f deploy_cm.yaml
    kubectl exec -ti nginx-dep-788c7849c7-7h47k -- ls /mnt/files/
    ```

### When the configmaps are updated, the pods are not updated automatically, for this you can use reloader -> [https://github.com/mario21ic/k8s-demos/tree/master/] reloader

**************************************

## K8s Secrets

1. Create file secret1.yaml

    ```console
    nano secret1.yaml
    ```

    ```yaml
    kind: Secret
    apiVersion: v1
    metadata:
      name: my-secret
    type: Opaque
    data:
      hello: SGVsbG8gZnJvbSBBdWxhVXRpbA==
      key: T1VSX0FQSV9BQ0NFU1NfS0VZCg==
      token: U0VDUkVUXzd0NDgzNjM3OGVyd2RzZXIzNAo=
    ```

    ```console
    kubectl apply -f secret1.yaml
    kubectl get secret
    kubectl describe secret my-secret
     ```

### Pod using the secret

1. Create file pod_secret.yaml

    ```console
    nano pod_secret.yaml
    ```

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-pod-secret
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        env:
          - name: HELLO
            valueFrom:
              secretKeyRef:
                name: my-secret
                key: hello
    ```

    ```console
    kubectl apply -f pod_secret.yaml
    kubectl get pods
    kubectl exec -ti nginx-pod-secret -- env
    ```

2. Dynamic: Recover the other two values (key and token).

3. Clean cluster:

    ```console
    kubectl delete all --all
    ```

    **************************************

## K8s | Volume NFS

### Pre-requisites

- Have an NFS Server.
  - NFS Server IP and directory.

1. Create file pod-nfs.yaml

    ```console
    nano pod-nfs.yaml
    ```

    ```yaml
    # Create a pod that reads and writes to the
    # NFS server via an NFS volume.

    kind: Pod
    apiVersion: v1
    metadata:
      name: pod-using-nfs
    spec:
      # Add the server as an NFS volume for the pod
      volumes:
        - name: nfs-volume
          nfs: 
            # URL for the NFS server
            server: 192.168.1.50 # Change this!
            path: /mnt/nfs-share/

      # In this container, we'll mount the NFS volume
      # and write the date to a file inside it.
      containers:
        - name: app
          image: alpine

          # Mount the NFS volume in the container
          volumeMounts:
            - name: nfs-volume
              mountPath: /var/nfs

          # Write to a file inside our NFS
          command: ["/bin/sh"]
          args: ["-c", "while true; do date >> /var/nfs/dates.txt; sleep 5; done"]
    ```

### PLUS | Create an NFS Server with K8s

1. Create file nfs-server.yaml

    ```console
    nano nfs-server.yaml
    ```

    ```yaml
    # Note - an NFS server isn't really a Kubernetes
    # concept. We're just creating it in Kubernetes
    # for illustration and convenience. In practice,
    # it might be run in some other system.

    # Create a service to expose the NFS server
    # to pods inside the cluster.

    kind: Service
    apiVersion: v1
    metadata:
      name: nfs-service
    spec:
      selector:
        role: nfs
      ports:
        # Open the ports required by the NFS server
        # Port 2049 for TCP
        - name: tcp-2049
          port: 2049
          protocol: TCP

        # Port 111 for UDP
        - name: udp-111
          port: 111
          protocol: UDP

    ---

    # Run the NFS server image in a pod that is
    # exposed by the service.

    kind: Pod
    apiVersion: v1
    metadata:
      name: nfs-server-pod
      labels:
        role: nfs
    spec:
      containers:
        - name: nfs-server-container
          image: cpuguy83/nfs-server
          securityContext:
            privileged: true
          args:
            # Pass the paths to share to the Docker image
            - /exports
    ```

Mas Info -> [https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-volumes-example-nfs-persistent-volume.html]
