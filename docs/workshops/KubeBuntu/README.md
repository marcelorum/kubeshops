# Ubuntu Lab 
> Como crear un único Pod en un Clúster de Kubernetes.

## Introduction

Cómo se puede crear un único Pod en un Clúster de Kubernetes en IBM Cloud? Un Pod es un grupo de uno o más contenedores, con recursos de red y almacenamiento compartidos y una especificación de cómo ejecutar los contenedores. Cuando un Pod ejecuta un solo contenedor, es como tener un entorno para un solo contenedor. Kubernetes administra los Pods en lugar de administrar los contenedores directamente.

En este workshop vamos a ver cómo implementar un Pod de Ubuntu en Kubernetes en IBM Cloud, con el objetivo de tener un Laboratorio para pruebas. Dado que los Pods son desechables, no es recomendable para usar en Producción. Crearemos un contenedor usando la última imagen de Ubuntu en Docker Hub.

## Requisitos
- Una cuenta gratuita de IBM Cloud. Te podes [registrar acá](https://cloud.ibm.com/registration) si no tenes una aun.
- Un Cluster de Kubernetes gratuito en IBM Cloud. [Obtener acá.](https://cloud.ibm.com/kubernetes/catalog/create)
- Tener habilitados los comandos `ibmcloud` y `kubectl`. [Configurar CLI.](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)


## Tiempo estimado
El tiempo estimado que puede llevar este workshop es menos de 30 minutos

## Pasos
1. [Correr como Pod]()
2. [Correr como Implementación]()
3. [Acceder a la consola de Ubuntu]()
4. [Instalar un paquete de prueba]()

Se puede correr como Pod o como Implementación, debajo se detallan las dos opciones.

### 1. Correr como Pod:

  Contenido de archivo `ubuntu-as-pod.yaml` para la creación del Pod.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: ubuntu
    labels:
      app: ubuntu
  spec:
    containers:
    - name: ubuntu
      image: ubuntu:latest
      command: ["/bin/sleep", "3650d"]
      imagePullPolicy: IfNotPresent
    restartPolicy: Always
  ```

  Aplicar la creación del Pod:
```bash
kubectl apply -f ubuntu-as-pod.yaml
```
  Listar los Pods activos:
```bash
kubectl get pods
```
### 2. Correr como Implementación

  Contenido del archivo `ubuntu-deployment.yaml` para la implementación.
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: ubuntu
    name: ubuntu
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: ubuntu
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: ubuntu
      spec:
        containers:
        - image: ubuntu
          name: ubuntu
          command: ["/bin/sleep", "3650d"]
          resources: {}
  ```

  Desplegar la Implementación:
```bash
kubectl apply -f ubuntu-deployment.yaml
```
Listar los pods activos:
```bash
kubectl get pods
```
Listar todo para ver la Implementación y el Pod:
```bash
kubectl get all
```

### 3. Acceder a la Consola del Pod:


  Luego de confirmar que el Pod se está ejecutando, se puede acceder a la consola con el siguiente comando:
```bash
kubectl exec --stdin --tty ubuntu -- /bin/bash
```

### 4. PRUEBA: Instalar paquetes en un pod de Ubuntu

  Se puede usar la herramienta estándar de administración de paquetes `apt` de Ubuntu.

  El siguiente ejemplo instala `telnet` en el contenedor de Ubuntu.
```bash
root@ubuntu:/# apt update
root@ubuntu:/# apt install telnet
```
  Probar funcionamiento de `telnet`:
```bash
root@ubuntu:/# telnet 10.10.6.5   8080
```

## Resumen
En este workshop pudimos ver como correr una instancia de Ubuntu en un solo Pod o en un Pod con una Implementación. Una instacia de Ubuntu es muy útil para usar como Laboratorio de pruebas de paquetes o versiones diferentes de productos o aplicaciones. También sirve como base para luego avanzar en otros aspectos de Kubernetes, como volúmenes persistentes o servicios de red.
