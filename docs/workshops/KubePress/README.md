# WordPress en Kubernetes
> Desplegar WordPress en Kubernetes en IBM Cloud
## Introducción

En este workshop vamos a ver el potencial de un [Cluster de Kubernetes en IBM Cloud](https://www.ibm.com/cloud/kubernetes-service) y aprender a desplegar uno de los más conocidos frameworks de sitios webs como lo es [WordPress](https://wordpress.org/). Hay que tener en cuenta que el Cluster de Kubernetes de la cuenta gratuita de IBM Cloud expira a los 30 días, tiempo suficiente para poder realizar varias pruebas y testear la tecnología.

Cada componente se ejecuta en un contenedor o grupo de contenedores por separado. WordPress representa una aplicación típica de varios niveles y cada componente tendrá sus propios contenedores. Los contenedores de WordPress serán el nivel de frontend y el contenedor MySQL será el nivel de base de datos / backend para WordPress.

## Objetivos
- Crear y definir un Volumen Persistente.
- Crear un Secret para proteger información sensible.
- Crear un despliegue de WordPresss.
- Crear un despliegue de una base de datos MySQL.

## Requisitos
- Una cuenta gratuita de IBM Cloud. Te podes [registrar acá](https://cloud.ibm.com/registration) si no tenes una aun.
- Un Cluster de Kubernetes gratuito. [Obtener acá](https://cloud.ibm.com/kubernetes/catalog/create)
- Tener habilitados los comandos `ibmcloud` y `kubectl`. [Configurar CLI](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)

## Tiempo estimado
El tiempo estimado que puede llevar este workshop es menos de 30 minutos

## Pasos para Desplegar en IBM Cloud
1. [Configurar Secrets para MySQL](#1-configurar-secret-para-mysql)
2. [Crear volumenes persistentes locales](#2-crear-volumenes-persistentes-locales)
3. [Crear los servicios de despliegue para WordPress y MySQL](#3-crear-los-servicios-de-despliegue-para-wordpress-y-mysql)
4. [Acceder al link externo de WordPress](#Accessing-the-external-wordpress-link)

### 1. Configurar Secret para MySQL
Secrets en Kubernetes permiten almacenar y administrar información confidencial, como contraseñas y claves ssh, de forma segura y flexible.

Crear un Secret con el siguiente comando (cambiar por su contraseña):
```bash
kubectl create secret generic mysql-pass --from-literal=password='tucontraseniasegura'
```

### 2. Crear volumenes persistentes locales
Para guardar la información y los datos, más allá de la vida útil de cada pod de Kubernetes, necesitamos crear un volumen persistente para la base de datos MySQL y la aplicación de WordPress.

Contenido del archivo `local-volumes.yaml` para la creación del volume.
```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
name: local-volume-1
labels:
    type: local
spec:
capacity:
    storage: 20Gi
accessModes:
    - ReadWriteOnce
hostPath:
    path: /tmp/data/lv-1
persistentVolumeReclaimPolicy: Recycle
---
apiVersion: v1
kind: PersistentVolume
metadata:
name: local-volume-2
labels:
    type: local
spec:
capacity:
    storage: 20Gi
accessModes:
    - ReadWriteOnce
hostPath:
    path: /tmp/data/lv-2
persistentVolumeReclaimPolicy: Recycle
```

Crear el volumen persistente con el siguiente comando
```bash
kubectl create -f local-volumes.yaml
```
### 3. Crear los servicios de despliegue para WordPress y MySQL
Para instalar el volumen persistente en el almacenamiento local del cluster, usar los secrets y crear los servicios para MySQL y WordPress, vamos a usar los archivos de configuración `mysql-deployment.yaml` y `wordpress-deployment.yaml`

Contenido del archivo `mysql-deployment.yaml` para el servicio de mysql.
```yaml
apiVersion: v1
kind: Service
metadata:
name: wordpress-mysql
labels:
    app: wordpress
spec:
ports:
    - port: 3306
selector:
    app: wordpress
    tier: mysql
clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: mysql-pv-claim
labels:
    app: wordpress
spec:
accessModes:
    - ReadWriteOnce
resources:
    requests:
    storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
name: wordpress-mysql
labels:
    app: wordpress
spec:
selector:
    matchLabels:
    app: wordpress
    tier: mysql
strategy:
    type: Recreate
template:
    metadata:
    labels:
        app: wordpress
        tier: mysql
    spec:
    containers:
    - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
        valueFrom:
            secretKeyRef:
            name: mysql-pass
            key: password
        ports:
        - containerPort: 3306
        name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
        mountPath: /var/lib/mysql
    volumes:
    - name: mysql-persistent-storage
        persistentVolumeClaim:
        claimName: mysql-pv-claim
```

Contenido del archivo `wordpress-deployment.yaml` para el servicio de WordPress.
```yaml
apiVersion: v1
kind: Service
metadata:
name: wordpress
labels:
    app: wordpress
spec:
ports:
    - port: 80
selector:
    app: wordpress
    tier: frontend
type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: wp-pv-claim
labels:
    app: wordpress
spec:
accessModes:
    - ReadWriteOnce
resources:
    requests:
    storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
name: wordpress
labels:
    app: wordpress
spec:
selector:
    matchLabels:
    app: wordpress
    tier: frontend
strategy:
    type: Recreate
template:
    metadata:
    labels:
        app: wordpress
        tier: frontend
    spec:
    containers:
    - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
        value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
        valueFrom:
            secretKeyRef:
            name: mysql-pass
            key: password
        ports:
        - containerPort: 80
        name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
        mountPath: /var/www/html
    volumes:
    - name: wordpress-persistent-storage
        persistentVolumeClaim:
        claimName: wp-pv-claim
```

Crear los servicios para MySQL y WordPress.

```bash
kubectl create -f mysql-deployment.yaml
kubectl create -f wordpress-deployment.yaml
```

Para verificar que los los pods estén corriendo, usar el siguiente comando para ver el nombre de los pods:

```bash
kubectl get pods
```

Esto debería devolver la lista de pods del Cluster de Kubernetes.

```bash
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-57799cb7f8-prdxt         1/1     Running   0          31s
wordpress-mysql-5896755cf4-mq5q5   1/1     Running   1          40s
```

### 4. Acceder al link externo de WordPress
Para obtener la dirección IP de su cluster siga estos pasos:

Para ver el nombre del Cluster:
```bash
ibmcloud ks cluster ls
```
Para ver la IP Pública:
```bash
ibmcloud ks workers --cluster <nombre_del_cluster>
```
El resultado es similar a este:
```bash
OK
ID                                                       Public IP      Private IP      Flavor   State    Status   Zone    Version   
kube-c1upsl3d07j9adhlggm0-ibmkubeclou-default-00000000   169.57.43.40   10.131.75.187   free     normal   Ready
```
Para obtener el puerto:
```bash
kubectl get services wordpress
```

El resultado es similar a este:
```bash
NAME        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
wordpress   LoadBalancer   172.21.160.82   <pending>     80:30180/TCP   23s
```

**Felicitaciones!** Ahora podemos usar el link **http://[Public IP]:[PORT]** para acceder al sitio de WordPress.

> **Nota:** Para este ejemplo, el enlace quedaría de esta forma http://169.57.43.40:30180

## Resumen

En este workshop pudimos ver el potencial de Kuberneter en IBM Cloud y aprendimos a hacer un despliegue de WordPress con una base de datos MySQL, configurar los datos sensibles usando Secrets y generar volumenes persistentes para conservar la información.
