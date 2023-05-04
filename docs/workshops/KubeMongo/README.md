# PHP con MongoDB
Aplicación PHP con MongoDB en Kubernetes

## Introducción
En este tutorial vamos a ver cómo implementar una aplicación PHP tipo **Libro de Visitas** con varios niveles utilizando Kubernetes y Docker, con una instancia única de MongoDB para almacenar las  entradas del libro de visitas y varias instancias de frontend web.

## Requisitos
- Una cuenta gratuita de IBM Cloud. Te podes [registrar acá](https://cloud.ibm.com/registration) si no tenes una aun.
- Un Cluster de Kubernetes gratuito en IBM Cloud. [Obtener acá.](https://cloud.ibm.com/kubernetes/catalog/create)
- Tener habilitados los comandos `ibmcloud` y `kubectl`. [Configurar CLI.](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)

## Tiempo estimado
El tiempo estimado que puede llevar este tutorial es de 30 minutos

## Pasos
1. [Implementar MongoDB.]()
2. [Crear el Servicio de MongoDB]()
3. [Configurar y exponer la interfaz del libro de visitas]()
4. [Creando el Servicio Frontend]()
5. [EXTRA: Escalar el servicio Frontend]()

### Iniciar la base de datos de MongoDB
Creación de la implementación de MongoDB. El archivo `mongo-deployment.yaml` especifica un controlador que ejecuta una única réplica de MongoDB.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
    name: mongo
    component: backend
spec:
  selector:
    matchLabels:
      name: mongo
      component: backend
  replicas: 1
  template:
    metadata:
      labels:
        name: mongo
        component: backend
    spec:
      containers:
      - name: mongo
        image: mongo:4.2
        args:
          - --bind_ip
          - 0.0.0.0
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 27017
```

Para aplicar la implementación contenida en el archivo mongo-deployment.yaml, usar el siguiente comando:

```bash
kubectl apply -f mongo-deployment.yaml
```
Ver la lista de Pods y verificar que el Pod de MongoDB esté corriendo:
```bash
kubectl get pods
```
Para ver los registros de la implementación del Pod, usar el siguiente comando:
```bash
kubectl logs -f deployment/mongo
```
### Crear el Servicio de MongoDB
La aplicación de Libro de Visitas debe comunicarse con MongoDB para esribir la información. Se debe aplicar un servicio que defina la política de acceso para el tráfico de la información. Esto se define con el contenido del archivo `mongo-service.yaml` que se detalla a continuación:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    name: mongo
    component: backend
spec:
  ports:
  - port: 27017
    targetPort: 27017
  selector:
    name: mongo
    component: backend
```
Para aplicar el servicio con el archivo mongo-service.yaml usar el siguiente comando:
```bash
kubectl apply -f mongo-service.yaml
```
Ver la lista de Servicios y verificar que el Servicio de MongoDB esté corriendo:

```bash
kubectl get service
```

### Configurar y exponer la interfaz del libro de visitas
El Libro de Visitas tiene una interfaz web que atiende las solicitudes HTTP escritas en PHP. Está configurado para conectarse al Servicio de MongoDB para almacenar entradas del Libro de Visitas.

La implementación del Frontend del Libro de Visitas se detalla en el contenido del archivo `frontend-deployment.yaml` a continuación:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    name: guestbook
    component: frontend
spec:
  selector:
    matchLabels:
      name: guestbook
      component: frontend
  replicas: 3
  template:
    metadata:
      labels:
        name: guestbook
        component: frontend
    spec:
      containers:
      - name: guestbook
        image: marcelorum/gb-frontend:v5
        # image: gcr.io/google-samples/gb-frontend:v5
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
        ports:
        - containerPort: 80
```
> **Nota:** Se puede utilizar la imagen en Docker que he creado para este fin o el ejemplo desde gcr.io/google-samples/gb-frontend:v5

Aplicar la implementación de frontend con el archivo frontend-deployment.yaml:
```bash
kubectl apply -f frontend-deployment.yaml
```
Ver la lista de Pods y verificar que las tres réplicas de frontend se estén ejecutando:
```bash
kubectl get pods -l name=guestbook -l component=frontend
```

### Creando el Servicio Frontend
Para que el Libro de Visitas sea accedido externamente, tenemos que crear un servicio y exponer el puerto externamente. Esto lo podemos hacer aplicando el contenido del archivo `frontend-service.yaml` que se detalla a continuación:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    name: guestbook
    component: frontend
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    name: guestbook
    component: frontend
```
> Nota: La ventaja de estar usando Kubernetes en IBM Cloud, es que admite el uso de balanceadores de carga externos. Por eso usamos el siguiente componente `type: LoadBalancer`.

Aplicar el servicio con el archivo frontend-service.yaml:
```bash
kubectl apply -f frontend-service.yaml
```
Verificar que el Servicio de Frontend esté corriendo y ver el puerto:
```bash
kubectl get service frontend
```
Para obtener la IP Pública:
```bash
kubectl get nodes -o wide
```
**Felicitaciones!** Ahora podemos usar el enlace **http://[Public IP]:[PORT]** para acceder a la aplicación.


### EXTRA: Escalar el servicio Frontend
Puede escalar hacia arriba o hacia abajo según sea necesario porque sus servidores están definidos como un Servicio que usa un controlador de implementación.

Para escalar el número de Pods de Frontend usar el siguiente comando:
```bash
kubectl scale deployment frontend --replicas=5
```
Ver la lista de Pods y verificar el número de Pods que estan corriendo:
```bash
kubectl get pods
```
Para reducir la cantidad de Pods usar el siguiente comando:
```bash
kubectl scale deployment frontend --replicas=2
```
Ver la lista de Pods y verificar el número de Pods que estan corriendo:
```bash
kubectl get pods
```

**Cleaning up**

Al eliminar las implementaciones y los servicios, también se eliminan los pods en ejecución. Se pueden utilizar etiquetas para eliminar varios recursos con un solo comando.

Para eliminar todos los Pods, Implementaciones y Servicios, usar los siguientes comandos:
```bash
kubectl delete deployment -l name=mongo
kubectl delete service -l name=mongo
kubectl delete deployment -l name=guestbook
kubectl delete service -l name=guestbook
```

## Resumen / Conclusión
En este tutorial vimos como implementar una aplicación web en PHP aplicada desde una imagen de Docker Hub, conectada a un servicio de base de datos MongoDB implementada en este mismo ejercicio y conectada a través de un Servicio creado para tal fin. Además vimos como escalar y reducir el número de Pods de la aplicación Frontend. Por último, limpiamos toda la implementación.

## Enlaces Interesantes
- [Cómo crear una cuenta gratuita en IBM Cloud](https://cloud.ibm.com/docs/account?topic=account-account-getting-started)
- [Servicios de Kubernetes en IBM Cloud](https://www.ibm.com/cloud/kubernetes-service)
- [Cómo instalar el CLI de IBM Cloud](https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli)
- [Descripción general de kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
- [Introducción a Docker](https://docs.docker.com/get-started/)
