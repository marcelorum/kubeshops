# Desplegar con Docker
> Desplegar una aplicación desde una imagen propia de Docker en Kubernetes en IBM Cloud
## Introducción

En este workshop vamos a ver el potencial de Docker y Kubernetes usados conjuntamente, aprenderemos a crear y modificar una imágen de Docker, subirla al repositorio en Docker Hub y actualizarla. Luego vamos a usar esa imagen para hacer un despliegue en un Cluster de Kubernetes en IBM Cloud, revisar los pods y escalar la fuerza de trabajo. Por último vamos a exponer la aplicación para poder acceder de manera externa, verificar el funcionamiento y también haremos algunos cambios en el servicio para probar la actualización del mismo y que podamos seguir accediendo a la aplicación.

## Requisitos
- Sistema de gestión de paquetes de Node.js `npm`. [Obtener npm/Node.js](https://www.npmjs.com/get-npm)
> **Nota:** En caso de no querer instalar Node.js, se puede usar el contenido del archivo [KubeNode.zip](KubeNode.zip). Descomprimirlo en el mismo directorio.
- Docker y comando `docker` [Obtener Docker](https://www.docker.com/get-started)
- Acceso a Docker Hub (en el caso de querer usar su propia imagen). [Docker Hub](https://hub.docker.com/)
> **Nota:** En caso de no querer usar una imagen propia, se puede usar la imagen marcelorum/node-hello-app creada para este fin.
- Una cuenta gratuita de IBM Cloud. Te podes [registrar acá](https://cloud.ibm.com/registration) si no tenes una aun.
- Un Cluster de Kubernetes gratuito. [Obtener acá](https://cloud.ibm.com/kubernetes/catalog/create)
- Tener habilitados los comandos `ibmcloud` y `kubectl`. [Configurar CLI](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)

## Tiempo estimado
El tiempo estimado que puede llevar este workshop es de 30 a 60 minutos con todos los requerimientos.

## Este workshop se divide en tres secciones
1. [Node App](#1-node-app)
2. [Docker](#2-docker)
3. [Kubernetes](#3-kubernetes)

### 1. Node App

> **Nota:** En caso de usar el contenido del archivo [KubeNode.zip](KubeNode.zip), saltar este paso.

1. En el directorio del proyecto, crear un paquete vacío e inicializar Node.js:
```bash
npm init -y
```

2. Instalar el modulo para aplicaciones web de Node.js _Express_:
```bash
npm install express
```

3. Dejar en el mismo directorio el archivo `index.js`

    Contenido del archivo `index.js` para la implementación.
    ```java
    const express = require('express')
    const os = require('os')

    const app = express()
    app.get('/', (req, res) => {
            res.send(`Hola a todos desde ${os.hostname()}!`)
    })

    const port = 3000
    app.listen(port, () => console.log(`listening on port ${port}`))
    ```

### 2. Docker

#### Caso 1: Con imagen propia

1. Vamos a usar el archivo `dockerfile` que tiene que estar en el mismo directorio que se está utilizando.

    Contenido del archivo `dockerfile` para la implementación.
    ```bash
    FROM node:13-alpine

    WORKDIR /app

    COPY package.json ./

    RUN npm install --production

    COPY . .

    EXPOSE 3000

    CMD node index.js
    ```

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
docker stop $(docker ps -a -q)
```

6. Subir la imagen al repositorio en Docker Hub:
```bash
docker push <dockeruser>/node-hello-app
```

#### Caso 2: Con otra imagen

1. Vamos a usar la imagen `marcelorum/node-hello-app` creada para este fin.

    Traer la imágen del registro:
    ```bash
    docker pull marcelorum/node-hello-app
    ```

2. Correr el contenedor localmente para probar:
```bash
docker run --rm -d -p 3000:3000 marcelorum/node-hello-app
```

3. Revisar los contenedores que están corriendo:
```bash
docker ps
```

    > **Nota:** Se puede revisar en http://localhost:3000

4. Terminar las tareas de contenedor:
```bash
docker stop <CONTAINER_ID>
docker stop $(docker ps -a -q)
```

#### EXTRA: Prueba de actualización:
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
```bash
kubectl create deployment --image marcelorum/node-hello-app node-app
```

    > **Nota:** En caso de usar la imagen ya creada marcelorum/node-hello-app.

3. Exponer el despliegue como una replica NodePort:
```bash
kubectl expose deployment node-app --type=NodePort --port 3000
```

4. Revisar el servicio creado y el puerto asignado:
```bash
kubectl get services
```

5. Para obtener la IP Pública:
```bash
kubectl get nodes -o wide
```
  **Felicitaciones!** Ahora podemos usar el enlace **http://[Public IP]:[PORT]** para acceder al servicio.

#### EXTRA: Prueba de esfuerzo:
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

#### EXTRA: Prueba de modificación del Servicio:
  - Editar el servicio:
  ```bash
  kubectl edit service node-app
  ```
  > **Nota:** Se sale del editor con :wq

  - Reemplazar el puerto: `port: 3000` por `port: 80`
  - Reemplazar el tipo: `type: NodePort` por `type: LoadBalancer`
  - Verificar que el servicio se haya actualizado:
  ```bash
  kubectl get service
  ```
  - Usar el enlace **http://[Public IP]:[PORT]** para verificar.

## Resumen
En este workshop aprendimos a crear una imagen de Docker y subirla al repositorio, luego usar esa imagen para hacer un despliegue en un Cluster de Kubernetes en IBM Cloud. Además vimos el comportamiento al realizar modificaciones en la imagen de Docker y en los Servicios de Kubernetes y también a escalar la aplicación con más réplicas.
