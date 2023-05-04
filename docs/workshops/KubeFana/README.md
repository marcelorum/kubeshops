# Monitoreo en Kubernetes
Crear una instacia de Grafana en Kubernetes y acceder externamente

## Introducción

Grafana es una herramienta de panel de código abierto que se puede utilizar para mostrar métricas de diferentes sistemas. Se puede integrar con una variedad de fuentes de datos como Prometheus, InfluxDB Stackdriver, etc.

En este tutorial vamos a ver cómo configurar un panel de Grafana en un Clúster de Kubernetes en IBM Cloud, usando Prometheus como herramienta de análisis de datos. Los siguientes pasos nos guiarán para configurar una instancia de Grafana en funcionamiento. Además veremos como crear un namespace para nuestro proyecto.

## Requisitos
- Una cuenta gratuita de IBM Cloud. Te podes [registrar acá](https://cloud.ibm.com/registration) si no tenes una aun.
- Un Cluster de Kubernetes gratuito en IBM Cloud. [Obtener acá.](https://cloud.ibm.com/kubernetes/catalog/create)
- Tener habilitados los comandos `ibmcloud` y `kubectl`. [Configurar CLI.](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)

## Tiempo estimado
El tiempo estimado que puede llevar este tutorial es de 30 minutos.

## Pasos

1. Implementar el servicio de Grafana en Kubernetes

  Crear un nuevo proyecto para el despliegue:
```bash
kubectl create namespace monitoring
```
  Implementar Grafana en el namespace que acabamos de crear, usando la última imagen de Docker Hub:
```bash
kubectl create deployment grafana -n monitoring --image=docker.io/grafana/grafana:latest
```
  Esto implementa Grafana en el cluster y lo inicializa, para ver el estado de la implementación usar:
  ```bash
kubectl get deployments -n monitoring
```

2. Exponer el servicio de Grafana usando NodePort

  Tenemos que exponer el servicio via NodePort para que podamos acceder externamente, esto se logra con el siguiente comando:
```bash
kubectl -n monitoring expose deployment grafana --type="NodePort" --port 3000
```
  Esto crea el servicio y expone el puerto 3000, que es el puerto por defecto para Grafana. Para ver si el servicio ha sido expuesto correctamente:
```bash
$ kubectl get service -n monitoring
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana   NodePort   172.21.61.27   <none>        3000:32309/TCP   6d19h
```
  Este y el siguiente comando pueden ser usados para encontrar el puerto externo que ha sido expuesto internamente, en este caso es el puerto `32309`.
```bash
  kubectl describe service -n monitoring|grep NodePort
```
  Para obtener la IP Pública:

  ```bash
  $ kubectl get nodes -o wide
  NAME            STATUS   ROLES    AGE   VERSION       INTERNAL-IP     EXTERNAL-IP
  10.131.75.187   Ready    <none>   17d   v1.19.9IKS   10.131.75.187   169.57.53.40
  ```
  En este ejemplo, la IP pública es: `169.57.53.40`


3. Acceder al panel web de Grafana

  **Felicitaciones!** Ahora podemos usar el enlace **http://[Public IP]:[PORT]** para acceder al servicio de la instacia de Grafana en nuestro Clúster de Kubernetes.

  Para este ejemplo, la URL es: `http://169.57.53.40:32309`. Esto abre la página inicial de Grafana que requeire acceso. El usuario y contraseña por defecto es `admin/admin`. En el primer ingreso es necesario cambiar la contraseña.

4. Probar Grafana

  Necesitamos probar la instancia de Grafana para ver si funciona correctamente. La manera más sencilla de hacer esto es usar "theTestData DB" la cual nos da ejemplos de visualización de datos. Para esto, en nuestro panel de Grafana hacer lo siguiente:

  Hacer click en "Create your first data source" y elegir "TestData DB". Click en "Save and test".

  Hacer click en "Create a new dashboard" para crear un nuevo Panel.

  Para ver el panel con información, hacer click en "heat map" o "graph".

  Hay muchas plantillas disponibles para crear paneles con funcionalidades que aplican para una variada cantidad de ambientes. Se pueden encontrar plantillas [acá.](https://grafana.com/grafana/dashboards?search=kubernetes).

5. Monitorear nuestro Clúster de Kubernetes con nuestra instancia de Grafana

    1. Obtener el ID de [esta plantilla pública](https://grafana.com/grafana/dashboards/8588) haciendo Click en "Copy ID to Clipboard".
    2. En nuestro panel de Grafana hacer click en Import.
    3. Ingresar el ID obtenido en el punto 1.
    4. Hacer click en Load.
    5. Hacer click en import para importar el panel. Esto nos llevará al panel con las métricas de nuestro Clúster.


## Resumen
Grafana es una herramienta muy ligera y poderosa para visualización de cargas de trabajo. Se puede integrar con varias herramientas de monitoreo y se adapta en diferentes escenarios. Esto incluye sistemas en la nube, entornos de red y contenedores.