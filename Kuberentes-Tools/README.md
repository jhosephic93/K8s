## TOOLS FOR KUBERNETES

## OPENLENS

- Install on MacOS

    ```console
    brew install openlens --casks
    ```

- Repository -> <https://github.com/MuhammedKalkan/OpenLens>

## HELM

- Web => <https://helm.sh/> and Chats => <https://artifacthub.io/>
- Administrador de paquetes para K8s
- Permite buscar, instalar y modificar paquetes completos
- Helm tiene 3 partes
  - **CHART** => Plantillas de recursos Kubernetes.
    - Definición: Conjunto de archivos YAML preconfigurados.
  - **REPOSITORY** => Almacén de charts Helm.
    - Definición: Servidor para alojar y compartir charts.
  - **RELEASE** => Instancia de chart desplegada.
    - Definición: Implementación específica de un chart.

1.  Add repository and list

    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo list
    ```

2.  Search chart

    ```bash
    helm search repo bitnami #Search in repository Bitnami
    helm search repo wordpress #Search all repositories
    helm search repo bitnami/wordpress
    ```

### INSTALL CHART WITH HELM

1. Install WordPress on Kubernetes with HELM of stable repository.

    ```console
    helm install bitnami/wordpress --generate-name
    #OR
    helm install --set name=v1 bitnami/wordpress
    ```

2. Inspect Chart of WordPress

    ```console
    helm show chart bitnami/wordpress
    helm show readme bitnami/wordpress
    ```

### CUSTOM CHART WITH HELM

1. OPTION01 | Custom Chart of WordPress

    ```console
    helm show values bitnami/wordpress
    helm show values bitnami/wordpress > config.yaml
    vim config.yaml
    helm install -f config.yaml bitnami/wordpress --generate-name
    ```

2. OPTION02 | Custom Chart of WordPress

    ```console
    helm show values bitnami/wordpress
    vim install.yaml
    wordpressUsername: admin
    wordpressPassword: admin
    helm install mywordpress -f install.yaml bitnami/wordpress
    ```

### LIST CHART INSTALLED & DELETE

    ```console
    helm list
    helm list --all-namespaces
    helm uninstall [name-release]
    ```