## PRÁCTICA 6:

## CREACIÓN DE UN CLÚSTER DE KUBERNETES Y DESPLIEGUE DE APLICACIONES

## Descripción

En esta práctica, el alumno aprenderá a desplegar un clúster de Kubernetes en Google Cloud y a desplegar diversas aplicaciones sobre él. También se realizarán tareas sencillas de monitorización.

## Objetivos

  * Afianzar los conocimientos adquiridos en la gestión de recursos virtuales en proveedores de Cloud.
  * Aprender a realizar tareas de desarrollador y de administrador en entornos Cloud.

## Documentos y ficheros proporcionados

Cada alumno dispone de una cuenta de Google Cloud que puede solicitar usando un enlace previamente distribuido. Esto debe realizarse antes de la ejecución de la práctica.

## Actividades a desarrollar

La práctica se divide en dos partes: primero el despliegue de una aplicación y luego su monitorización.

### Parte 1: Despliegue de una aplicación en Google Cloud System

En esta parte de la práctica, el alumno creará un clúster de Kubernetes en Google Cloud. Se editará una memoria donde se reflejarán los pasos y reflexiones indicados.

1.  Acceda al portal de Google Cloud System e inicie sesión con la cuenta creada previamente. Acceda a la consola: [https://console.cloud.google.com/home](https://console.cloud.google.com/home).
    Una vez conectado al portal, cree un **proyecto**, por ejemplo, con el nombre `practica`, sin acentos. Observe que se le asocia un código que consta del nombre más un número. Esto se utilizará más adelante.
    Acceda a la creación de un clúster de Kubernetes: Seleccione las opciones por defecto y cambie la región a `europesouthwest1`. **Indique en la memoria por qué cree que se ha seleccionado esta región.**
    Espere unos instantes hasta que se cree el clúster. Nota: también se puede crear con la opción "**Standard, you manage your cluster**".
    **Indique qué diferencias cree que existen entre el uso de `docker-compose` (parte opcional de la práctica 5) y lo realizado en esta práctica.**
    Puede acceder a Compute Engine -\> Instancias de VM. **Explique qué ha aparecido en esta vista.**

2.  Regrese a Kubernetes Engine -\> Clusters.

3.  Deslice la descripción del clúster hasta que aparezcan tres puntos verticales y seleccione **Conectar**, luego seleccione **Ejecutar en Cloud Shell**. Una vez abierta, utilícela para enviar los comandos al clúster. Ejecute la línea que se indica (una vez que el clúster esté arrancado, lo cual se puede ver en el campo **Estado**).
    Se puede verificar con:

    ```bash
    kubectl get nodes
    ```

4.  Ahora ya tenemos un clúster de máquinas donde podemos desplegar **Pods**, formados por contenedores Docker. Para ello, necesitamos los ficheros que describen la aplicación.
    **Realice una captura e indique de cuántas máquinas virtuales está formado el clúster.**

5.  Podemos ir a la página de control del clúster (nótese que se puede ver como máquinas virtuales o como el clúster completo según la página que visitemos): [https://console.cloud.google.com/kubernetes](https://console.cloud.google.com/kubernetes) y verificar que está funcionando correctamente.

6.  Despliegue de una aplicación sencilla. Vamos a usar una aplicación web muy sencilla escrita en **Node.js**:

    ```js
    var http = require('http');

    var handleRequest = function(request, response) {

    console.log('Received request for URL: ' + request.url);

    response.writeHead(200);

    response.end('Hello World!');

    };

    var www = http.createServer(handleRequest);

    www.listen(8080);
    ```

    Que ya está disponible como contenedor Docker con el siguiente `Dockerfile`: [https://raw.githubusercontent.com/kubernetes/website/main/content/uk/examples/minikube/](https://raw.githubusercontent.com/kubernetes/website/main/content/uk/examples/minikube/) Dockerfile

    ```dockerfile
    FROM node:6.14.2

    EXPOSE 8080

    COPY server.js .

    CMD node server.js
    ```

    Este contenedor ya está disponible y lo usaremos para el despliegue en nuestro clúster:

    ```bash
    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53 -- /agnhost netexec --http-port=8080
    ```

    Podemos verlo con los siguientes comandos:

    ```bash
    kubectl get deployments

    kubectl get pods

    kubectl get events

    kubectl config view
    ```

    Espere hasta que el Pod esté operativo. **Indique qué nos muestran estos comandos.**
    Lo desplegaremos en un Pod con:

    ```bash
    kubectl expose deployment hello-node --type=LoadBalancer --port=8080
    ```

    Para acceder al servicio:

    ```bash
    kubectl get services
    ```

    Y para escalarlo:

    ```bash
    kubectl scale deployments/hello-node --replicas=4

    kubectl get pods

    kubectl get deployments
    ```

7.  Verifique cuál es el punto de acceso al servicio usando de nuevo el siguiente comando y conéctese al servidor. **Indique en la memoria cuál es el punto de acceso.**

    ```bash
    kubectl get services
    ```

8.  **Indique cuántos contenedores están ejecutando el servidor y realice una comparación con el escenario planteado en la práctica creativa 1.**

-----

## Parte 2: Despliegue de una apliación basada en microservicios.

Como ejemplo vamos a utilizar una tienda online formada por múltiples microservicios. Vea la descripción en:

[https://github.com/GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)

Para instalarla, ejecute:

```bash
git clone https://github.com/GoogleCloudPlatform/microservices-demo

cd microservices-demo

kubectl apply -f ./release/kubernetes-manifests.yaml

kubectl get pods
```

**Indique cuántos servicios y Pods se han configurado y arrancado.**

A continuación, obtenga la dirección IP del servicio que le permite acceder a la tienda:

```bash
kubectl get service/frontend-external
```

Acceda al servidor web que está en la IP externa con su navegador.

**Indique cuántas máquinas virtuales forman el clúster y cuántos Pods tenemos arrancados.**

Para demostrar la capacidad de escalado de Kubernetes, procederemos a generar carga sobre la tienda y a autoescalar los componentes necesarios mediante el autoescalador horizontal de Pods (HPA). HPA necesita un servidor de métricas que le proporcione información sobre el estado de los pods. En el caso de GKE, este viene instalado por defecto.
La tienda de ejemplo usada cuenta con un generador de tráfico que nos permitirá generar carga en el sistema. El generador de tráfico es un Pod llamado `loadgenerator-<id>` y consiste en un *script* de *Locust* que simula usuarios accediendo a la tienda *online*. Para aumentar la carga del servicio, podemos aumentar el número de usuarios simulados y la tasa de generación de peticiones.

Para ello, acceda al fichero `release/kubernetes-manifests.yaml`, busque el *deployment* `loadgenerator` y cambie los valores de las variables `USERS` y `RATE` a `1000` y `100` respectivamente.

Además, debemos subir los recursos asignados al Pod `loadgenerator` para que pueda generar la carga necesaria. Cambiaremos los valores de `requests` y `limits` de CPU a `2000m` y la memoria a `2048Mi`.

Ejecute otra vez el siguiente comando para actualizar los valores:

```bash
kubectl apply -f ./release/kubernetes-manifests.yaml
```

A continuación, monitorizaremos qué *deployment* requiere más recursos para soportar la carga. Para ello, usaremos el comando:

```bash
watch kubectl get pods
```

También podemos ver qué peticiones genera el `loadgenerator` y cuáles están siendo atendidas correctamente con el siguiente comando:

```bash
kubectl logs loadgenerator-<id-pod> --tail 20
```

Cuando observe que algún despliegue deja de estar disponible, ejecute el comando:

```bash
kubectl autoscale deployment <deployment-name> --min 1 --max 10
```

El comando anterior creará un autoescalador horizontal de Pods (HPA) para el despliegue indicado, con un mínimo de 1 réplica y un máximo de 10 réplicas.

Para monitorizar el estado del HPA, podemos usar el comando:

```bash
kubectl get hpa
```

Si muestra `unknown_cpu`, espere unos minutos hasta que se inicie el servidor de métricas y vuelva a ejecutar el comando.

**¿Cuántos Pods se han desplegado del servicio que se ha autoescalado?**

-----

**ENTREGA:** Se solicita entregar el texto y los comandos que aparecen en la consola, así como los comentarios pedidos a lo largo de la práctica.

**IMPORTANTE:** Una vez finalizada la práctica, destruya las máquinas virtuales para evitar gastar recursos en ellas. Para ello, utilice la interfaz de máquina virtuales (destruir todas) o la de clústeres de Kubernetes (destruir el único clúster).

