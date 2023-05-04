# Kubernetes a fondo
> Kubernetes en lo Profundo, tres elementos importantes
## Introducción
En este tutorial vamos a ver tres elementos que son muy importantes para ir profundizando en el uso de Kubernetes, más allá de su uso para pruebas de laboratorio, pensando en un uso productivo y el aprovechamiento del potencial de esta herramienta. Vamos a ver cómo configurar namespaces y poder ordernar proyectos, configurar entornos multi-containers y crear volúmenes persistentes para poder retener información de los Pods.

## Requisitos
- Una cuenta gratuita de IBM Cloud. [Registrar acá.](https://cloud.ibm.com/registration)
- Un Cluster de Kubernetes gratuito en IBM Cloud. [Obtener acá.](https://cloud.ibm.com/kubernetes/catalog/create)
- Tener habilitados los comandos `ibmcloud` y `kubectl`. [Configurar CLI.](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)

## Tiempo estimado
El tiempo estimado que puede llevar este tutorial es de 30 a 45 minutos con todos los requerimientos.

## Este tutorial se divide en tres secciones
1. [Cómo crear un namespace.]()
2. [Cómo crear un Pod multi-container]()
3. [Cómo crear un Volumen para Almacenamiento persistente]()

### 1. Cómo crear un namespace
Los namespaces son muy útiles para una mejor gestión de Kubernetes. En este workshop veremos como crear un nuevo namespace e implementar un pod en él.

En Kubernetes, un namespace se puede usar en entornos donde varios usuarios trabajan en diferentes equipos o proyectos. Con el uso de namespaces es posible dividir los recursos entre los usuarios sin que se produzca una colisión de nombres.

Namespaces son una especie de clúster virtual, parecido a tener varias máquinas virtuales y podemos tener varios namespaces en el mismo Clúster.

Al crear un clúster de Kubernetes, se inician al menos tres espacios de nombres básicos:

- default: se utiliza para implementaciones sin namespace asignado
- kube-system: se utiliza para todo lo relacionado con el sistema Kubernetes
- kube-public: reservado solo para uso del sistema

Para ver la lista de namespaces actuales usar el comando:
```bash
kubectl get namespaces
```
Ejemplo de mi Clúster de prueba:
```bash
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   18d
ibm-cert-store    Active   18d
ibm-operators     Active   18d
ibm-system        Active   18d
kube-node-lease   Active   18d
kube-public       Active   18d
kube-system       Active   18d
```

Vamos a crear un namespace llamado _staging_ corriendo el siguiente comando:
```bash
kubectl create namespace staging
```
Vamos a implementar un Pod NGINX en el nuevo namespace `staging` con el siguiente comando:
```bash
kubectl run nginx --image=nginx --namespace=staging
```
Para ver si el Pod se ha implementado correctamente vamos a correr el siguiente comando:
```bash
kubectl get pods --namespace=staging
```
Nuestro Pod NGINX se está ejecutando.

#### EXTRA: Cómo configurar el namespace predeterminado

Para usar un namespace específico como predeterminado, para no tener que usar la opción `--amespace=NAMESPACE` en los comandos de implementación, usar el siguiente comando:
```bash
kubectl config set-context --current --namespace=NAMESPACE
```
> **Nota:** NAMESPACE es el nombre del namespace que va a utilizar como predeterminado.

----

### 2. Cómo crear un Pod multi-container
Para crear un Pod multi-container vamos a utilizar el siguiente archivo `multi-container-pod.yml` con el siguietne contenido:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: container-1
    image: nginx
    ports:
    - containerPort: 80  
  - name: container-2
    image: alpine
    command: ["watch", "wget", "-qO-", "localhost"]
```
Este archivo tiene una definición para 2 contenedores que compartirán la misma red, recursos y volúmen.
Para crear un Pod de contenedores múltiples usar el siguiente comando:
```bash
kubectl create -f multi-container-pod.yml
```
Para ver los pods que estan corriendo, usar el siguiente comando:
```bash
kubectl get pods
```
Para ver el detalle del Pod:
```bash
kubectl describe pod multi-container-pod
```
Acá vamos a poder ver los detalles de los dos contenedores que contiene. Los contenedores contenedor-1 y contenedor-2. Ambos pertenecen al mismo grupo.

Para verificar los registros de un contenedor en particular, puede agregar el nombre del contenedor al comando:
```bash
kubectl logs multi-container-pod container-1
```

Si el nombre del contenedor no se proporciona en el comando, aparece un error en el que se le indica que especifique el nombre del contenedor. Se puede especificar el nombre de un único contenedor y no se pueden especificar varios contenedores en el comando para obtener los registros, _los siguientes dos comandos fallan_:
```bash
kubectl logs multi-container-pod
kubectl logs multi-container-pod container-1 container-2
```
El comando para iniciar sesión en el pod no funciona cuando hay varios contenedores dentro de un solo pod, _el siguiente comando falla_:
```bash
kubectl exec -it multi-container-pod /bin/bash
```
Para iniciar sesión en el contenedor en particular, necesitamos especificar el nombre del contenedor en el comando, además de que no se puede iniciar sesión en 2 pods al mismo tiempo:
```bash
kubectl exec -it multi-container-pod -c container-1 /bin/bash
```
Cuando el Pod ya no sea necesario, se puede eliminar y limpiar con el siguiente comando:
```bash
kubectl delete pod multi-container-pod
```

----

### 3. Configura un Volume para Almacenamiento persistente con un Pod

Vamos a ver cómo configurar un Pod para usar un Volúmen como almacenamiento. Los archivos de un contenedor existen mientras el Contenedor exista. Cuando un Contenedor es destruido o reiniciado, los cambios realizados se pierden. Para preservar datos se puede usar un Volume.

Va a ser un Pod que ejecuta un único Contenedor. El Volume es de tipo emptyDir y se va a mantener incluso cuando el Contenedor sea destruido y reiniciado. `volumen.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

Implementar el Pod:
```bash
kubectl apply -f volumen.yaml
```
Ver los Pods que están corriendo:
```bash
kubectl get pods
```
Vamos a inicar sesión en el Contenedor:
```bash
kubectl exec -it redis -- /bin/bash
```
Vamos a crear un archivo de prueba en /data/redis:
```bash
$ cd /data/redis/
$ echo Hola > prueba.txt
```
Vamos a listar los procesos en ejecución:
```bash
$ apt-get update
$ apt-get install procps
$ ps aux
```
Ahora debemos matar el proceso de Redis `redis-server`:
```bash
$ kill <pid>
```
>Donde <pid> es el ID de proceso (PID) de Redis.

Viendo los Pods con `kubectl get pods` podemos observar los cambios en el Pod de Redis. RESTARTS cuenta 1. El Contenedor ha sido destruido y reiniciado.

Vamos a inicar sesión en el Contenedor nuevamente:
```bash
kubectl exec -it redis -- /bin/bash
```
Vamos a ir a /data/redis para verifica que el archivo de prueba aun existe:
```bash
$ cd /data/redis/
$ ls
```
Cuando el Pod ya no sea necesario, se puede eliminar y limpiar con el siguiente comando:
```bash
kubectl delete pod redis
```

## Resumen / Conclusión
En este workshop vimos como ordenar el trabajo en proyectos con el uso de namespaces, aprendimos cómo crear un pod de contenedores múltiples, obtener registros de un contenedor en particular e iniciar sesión en un solo contenedor. Además vimos como crear un almacenamiento local proporcionado por emptyDir, Kubernetes soporta diferentes tipos de soluciones de almacenamiento por red y se encarga de todos los detalles, tal como montar y desmontar los dispositivos en los nodos del clúster.
