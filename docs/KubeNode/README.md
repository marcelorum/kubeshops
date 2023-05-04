# Desplegar una aplicación desde una imagen propia de Docker en Kubernetes en IBM Cloud

## Introducción

En este tutorial vamos a ver el potencial de Docker y Kubernetes usados conjuntamente, aprenderemos a crear y modificar una imágen de Docker, subirla al repositorio en Docker Hub y actualizarla. Luego vamos a usar esa imagen para hacer un despliegue en un Cluster de Kubernetes en IBM Cloud, revisar los pods y escalar la fuerza de trabajo. Por último vamos a exponer la aplicación para poder acceder de manera externa, verificar el funcionamiento y también haremos algunos cambios en el servicio para probar la actualización del mismo y que podamos seguir accediendo a la aplicación.

## Requisitos
- Sistema de gestión de paquetes de Node.js `npm`. [Obtener npm/Node.js!](https://www.npmjs.com/get-npm)
- Docker y comando `docker` [Obtener Docker](https://www.docker.com/get-started)
- Acceso a Docker Hub (en el caso de querer usar su propia imagen). [Docker Hub](https://hub.docker.com/)
- Una cuenta gratuita de IBM Cloud. Te podes [registrar acá](https://cloud.ibm.com/registration) si no tenes una aun.
- Un Cluster de Kubernetes gratuito. [Obtener acá](https://cloud.ibm.com/kubernetes/catalog/create)
- Tener habilitados los comandos `ibmcloud` y `kubectl`. [Configurar CLI](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)


## Tiempo estimado
El tiempo estimado que puede llevar este tutorial es de 30 minutos a 60 minutos con todos los requerimientos.

## Este tutorial se divide en tres secciones
1. [Node App](#1-node-app)
2. [Docker](#2-docker)
3. [Kubernetes](#3-kubernetes)

### 1. Node App
1. En el directorio del proyecto, crear un paquete vacío e inicializar Node.js:
```bash
npm init -y
```

2. Instalar el modulo para aplicaciones web de Node.js _Express_:
```bash
npm install express
```

3. Dejar en el mismo directorio el archivo `index.js`

> **Nota:**  Si no se puede o quiere  instalar Node.js, se puede usar el contenido del archivo KubeNode.zip.
Descomprimirlo en el mismo directorio.

### 2. Docker
1. Vamos a usar el archivo `dockerfile` que tiene que estar en el mismo directorio que se está utilizando.

2. Construir la imagen Docker:

  ```bash
  docker build -t <dockeruser>/node-hello-app .
  ```

3. Correr el contenedor localmente para probar:
```bash
docker run --rm -d -p 3000:3000 <dockeruser>/node-hello-app
```

4. Revisar los contenedores que están corriendo:
```bash
docker ps
```

  > **Nota:** Se puede revisar en http://localhost:3000

5. Terminar las tareas de contenedor:
```bash
docker stop <CONTAINER_ID>
```

6. Subir la imagen al repositorio en Docker Hub:
```bash
docker push <dockeruser>/node-hello-app
```

  ##### EXTRA: Prueba de actualización:
  - Editar el archivo `index.js` y reemplazar la palabra _Hola_ por la palabra _Hello_.
  - Reconstruir la imagen y prestar atención del uso de las capas anteriores:
  ```bash
  docker build -t <dockeruser>/node-hello-app .
  ```

### 3. Kubernetes
1. Ver los nodos que se están ejecutando:
```bash
kubectl get nodes
```

2. Crear un despliegue usando la imagen que creamos:
```bash
kubectl create deployment --image <dockeruser>/node-hello-app node-app
```

  > **Nota:** Si no se puede/quiere instalar Docker, se puede usar la imagen marcelorum/node-hello-app creada para este fin.

3. Exponer el despliegue como una replica NodePort:
```bash
kubectl expose deployment node-app --type=NodePort --port 3000
```

4. Revisar el servicio creado y el peurto asignado:
```bash
kubectl get services
```

5. Para obtener la IP Pública:
```bash
kubectl get nodes -o wide
```
  **Felicitaciones!** Ahora podemos usar el enlace **http://[Public IP]:[PORT]** para acceder al servicio.

  ##### EXTRA: Prueba de esfuerzo:
  - Listar los pods activos:
  ```bash
  kubectl get pods
  ```
  - Escalar hasta 3 replicas:
  ```bash
  kubectl scale deployment node-app --replicas 3
  ```
  - Listar los pods activos:
  ```bash
  kubectl get pods
  ```

  ##### EXTRA: Prueba de modificación del Servicio:
  - Editar el servicio:
  ```bash
  kubectl edit service node-app
  ```
    > **Nota:** Se sale del editor escribiendo :wq

  - Reemplazar el puerto: port: 3000 a port: 80
  - Reemplazar el tipo: type: NodePort con type: LoadBalancer
  - Verificar que el servicio se haya actualizado:
  ```bash
  kubectl get service
  ```
  - Usar el enlace **http://[Public IP]:[PORT]** para verificar.

## Resumen
En este tutorial aprendimos a crear una imagen de Docker y subirla al repositorio, luego usar esa imagen para hacer un despliegue en un Cluster de Kubernetes en IBM Cloud. Además vimos el comportamiento al realizar modificaciones en la imagen de Docker y en los Servicios de Kubernetes y también a escalar la aplicación con más réplicas.

## Enlaces Interesantes
- [IBM Cloud Kubernetes Service](https://www.ibm.com/cloud/kubernetes-service)
- [Installing the stand-alone IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli)
- [Overview of kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
- [Docker overview](https://docs.docker.com/get-started/)
- [Introduction to Node.js](https://nodejs.dev/learn)
