# Capabilities

- Two types:
  - Privileged.
    - No solo root, sino tambien acceso a modulos del kernel, y dispositivos del host.
  - Unprivileged

- Linux divide los privilegios tradicionalmente asociados con el superusuario en distintas unidades, conocidas como capacidades, que se pueden habilitar y deshabilitar de forma independiente. Las capacidades son un atributo por subproceso.  |  Linux divide los privilegios tradicionalmente asociados con el superusuario en distintas unidades, conocidas como capacidades, que se pueden habilitar y deshabilitar de forma independiente. Las son un atributo por capacidades del subproceso.

Que son los capabilities?

- Son ciertas directrices que nosotros le podamos dar a los containers para que puedan hacer una accion u otra, por ejemplo crear interface de red.

## Types Capabilities

- CAP_AUDIT_CONTROL
- CAP_AUDIT_READ
- CAP_AUDIT_WRITE
- CAP_BLOCK_SUSPEND
- CAP_CHOWN
- CAP_DAC_OVERRIDE
- CAP_DAC_READ_SEARCH
- CAP_FOWNER
- CAP_FSETID
- CAP_IPC_LOCK
- CAP_IPC_OWNER
- CAP_KILL
- CAP_LEASE
- CAP_LINUX_IMMUTABLE
- ....

## Docker Containers Privileged

1. Archivo de ejemplo para compartir drivers entre Host y Container:

    ```bash
    git clone https://github.com/mario21ic/linux-kernel-module-demo
    cd linux-kernel-module-demo/
    sudo apt update && sudo apt install kmod gcc make -y
    make all
    sudo insmod lkm_example.ko
    dmesg # Debe verse el mensaje de Hello
    sudo lsmod | grep example
    lkm_example            16384  0
    ```

2. Probar en container

    ```console
    $ docker run -ti --privileged --cap-add=ALL -v /dev:/dev ubuntu:18.04 bash
    # cat /proc/1/status | grep Cap #Debe salir “0000003fffffffff” porque tiene todos los     capabilities
    # apt update && apt install kmod -y
    # lsmod | grep lkm_example
    # rmmod lkm_example
    # ls -la /dev/
    # exit
    ```

3. Verificar que en host anfitrion ya no este instalado el kernel modulo:

    ```console
    sudo lsmod |grep lkm_example # Nota: Si da error al momento de descargar una docker image:
    docker logout
    rm -rf ~/.docker
    ```

## Docker Linux Capabilities

1. En linux crear una interface de red:

    ```console
    sudo ip link add dummy00 type dummy
    sudo ip link show
    ```

2. Crear container con capacidad de crear interfaces de red:

    ```console
    docker run -it --rm --cap-add=NET_ADMIN ubuntu:14.04 bash
    ip link show
    ip link add dummy01 type dummy
    ip link show
    cat /proc/1/status |grep Cap #Debe salir “00000000a80435fb” porque solo tiene el net    admin.
    exit
    ```

3. Crear container sin capacidad de crear interfaces de red:

    ```console
    docker run -ti --cap-add=ALL --cap-drop=NET_ADMIN ubuntu:14.04 bash
    ip link show
    ip link add dummy01 type dummy #RTNETLINK answers: Operation not permitted
    cat /proc/1/status |grep Cap #Debe salir “0000003fffffefff” que tiene todo menos net    admin cap
    exit
    ```

### More info

- <https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities>
- <https://man7.org/linux/man-pages/man7/capabilities.7.html>

## Kubernetes Privileged and Capabilities

1. Crear archivo pod-privileged.yaml con contenido:

    ```console
    nano pod-privileged.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
     name: pod-privileged
    spec:
     containers:
     - name: sec-ctx-3
       image: ubuntu:18.04
       command: [ "sh", "-c", "sleep 1h" ]
       securityContext:
         allowPrivilegeEscalation: true
    ```

2. Ejecutar

    ```console
    kubectl apply -f pod-privileged.yaml
    kubectl get pods
    kubectl exec -ti pod-privileged -- bash
    id
    apt update && apt install kmod -y
    ```

3. En otra consola agregar el modulo lkm_example

    ```console
    sudo insmod ./lkm_example.ko
    lsmod | grep lkm
    lsmod |grep lkm_example
    cat /proc/1/status |grep Cap
    ## Deben ser asi:
    CapPrm: 00000000a80425fb
    CapEff: 00000000a80425fb
    exit
    ```

4. Crear archivo pod-capabilities.yaml con contenido:

    ```console
    nano pod-capabilities.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-capabilities
    spec:
      containers:
      - name: sec-ctx-4
        image: busybox:latest
        command: [ "sh", "-c", "sleep 1h" ]
        securityContext:
          capabilities:
            add: ["NET_ADMIN", "SYS_TIME"]
    ```

5. Ejecutar

    ```console
    kubectl apply -f pod-capabilities.yaml
    kubectl get pods
    kubectl exec -ti pod-capabilities sh
    id
    ip link add dummy01 type dummy
    ip link show
    cat /proc/1/status |grep Cap
    ## Deben ser asi:
    CapPrm: 00000000aa0435fb
    CapEff: 00000000aa0435fb
    exit
    ```

- Nota: si desean ver los valores y su combinacion <https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h>
