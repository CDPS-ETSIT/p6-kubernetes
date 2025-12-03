# PRÁCTICA 6:
# CREACIÓN DE UN CLÚSTER DE KUBERNETES Y DESPLIEGUE DE APLICACIONES
## Descripción
En esta práctica el alumno aprenderá a desplegar un clúster de Kubernetes en Google Cloud y desplegar diversas aplicaciones sobre él. Se realizarán tareas sencillas de monitorización sobre él.
## Objetivos
- Afianzar los conocimientos adquiridos en gestión de recursos virtuales en proveedores de Cloud.
- Aprender a realizar tareas de desarrollador y de administrador en entornos Cloud.
## Documentos y ficheros proporcionados
Cada alumno dispone de una cuenta de Google Cloud que puede solicitar usando un enlace previamente distribuido. Esto es necesario realizarlo con anterioridad a la realización de la práctica.
## Actividades a desarrollar
La práctica se divide en dos partes: primero un despliegue de una aplicación y luego su monitorización.
### Parte 1: Google Cloud System
En esta parte de la práctica el alumno creará un clúster de Kubernetes en Google Cloud System. Se editará una memoria donde se reflejarán los pasos y reflexiones indicados.
1. Acceder al portal de Google cloud System y logearse con la cuenta creada previamente. Accederemos a la consola https://console.cloud.google.com/home
Una vez conectados acceder al portal crearemos un proyecto, por ejemplo, con el nombre práctica, sin acentos. Ver que le asocia un código que consta del nombre más un número. Esto se utilizará después. Accederemos a la creación de un clúster de Kubernetes: Seleccionaremos las opciones por defecto y cambiaremos la región a europesouthwest1 Indique en la memoria porqué cree que se ha seleccionado esta región.
Espere unos instantes hasta que se crea el clúster. Nota: también se puede crear con “standard you manage your cluster”
Indique que diferencias cree que existen entre el uso de docker-compose (parte opcional de la práctica 5) y lo realizado en esta.
Puede acceder a Compute-engine -> VM instances Explique qué es lo que ha aparecido en esta vista.

2. Regresamos a Kubernetes Engine -> Clusters

3. Deslizamos la descripción del clúster hasta que aparecen tres puntos verticales Y seleccionaremos connect y seleccionaremos run in cloud Shell. Una vez abierta la utilizaremos para dar los comandos al clúster. Ejecutando la línea que nos indica (una vez que este arrancado el clúster, se puede ver en el campo status)

Se puede verificar con :

```bash

kubectl get nodes

```
4. Ahora ya tenemos un clúster de ordenadores donde podemos desplegar Pods, formados por contenedores Docker. Para ello necesitamos los ficheros que describen la aplicación.
Realice una captura e indique de cuantas máquinas virtuales está formado el clúster.

5. Podemos ir a la página de control del clúster (nótese que se puede ver como las máquinas virtuales o como el clúster completo según la página que visitemos): https://console.cloud.google.com/kubernetes y verificar que está funcionando correctamente.

6. Despliegue de una aplicación sencilla. Vamos a usar una aplicación web muy sencilla escrita en node.js

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

Que ya estará disponible como contenedor Docker con el siguiente Dockerfile: https://raw.githubusercontent.com/kubernetes/website/main/content/uk/examples/minikube/ Dockerfile



```dockerfile

FROM node:6.14.2

EXPOSE 8080

COPY server.js .

CMD node server.js

```

Este contenedor ya está disponible y lo usaremos para el despliegue en nuestro cluster:

```bash

kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4

```
Podemos verlo con los siguientes comandos

```bash

kubectl get deployments

kubectl get pods

kubectl get events

kubectl config view

```

Esperar hasta que este el Pod operativo. Indique que nos muestran estos comandos. Lo desplegaremos en un pod con:

```bash

kubectl expose deployment hello-node --type=LoadBalancer -- port=8080

```
Para acceder al servicio

```bash

kubectl get services

```
y para escalarlo

```bash

kubectl scale deployments/hello-node --replicas=4

kubectl get pods

kubectl get deployments

```

7. Verifique cual es el punto de acceso al servicio usando de nuevo y conéctese al servidor. Indique en la memoria cual es el punto de acceso.



```bash

kubectl get services

```

8. Indique cuantos contenedores están ejecutando el servidor y realice una comparación con el escenario planteado en la práctica creativa 1

## Microservicios

Para ello vamos a utilizar una tienda online formada por múltiples microservicios, ver la descripción en:

https://github.com/GoogleCloudPlatform/microservices-demo

Para ello realizamos



```bash

git clone https://github.com/GoogleCloudPlatform/microservices-demo

cd microservices-demo

kubectl apply -f ./release/kubernetes-manifests.yaml

kubectl get pods

```

Una vez están activados todos los servicios, esperar y repetir hasta que todos estén activos. Indique cuantos servicios y pods se han configurado y arrancado.

Entonces obtener la dirección IP del servicio.



```bash

kubectl get service/frontend-external

```

Acceda al servidor web que está en la IP externa.

Indique cuantas máquinas virtuales forman el clúster y cuantos pods tenemos arrancados.

A continuación escalaremos los componentes necesarios mediante el autoescalador horizontal de pods (HPA).

Dependiendo del clúster será necesario instalar un servidor de métricas. En el caso de GKE, este ya está instalado.

La tienda de ejemplo cuenta con un generador de tráfico que nos permitirá generar carga en el sistema. El generador de tráfico es un pod llamado loadgenerator-<id> y consiste en un script de locust que simula usuarios accediendo a la tienda online. Para aumentar la carga del servicio, podemos aumentar el número de usuarios simulados y la tasa de generación de peticiones.

Para ello acceder fichero microservices-demo/release/release/kubernetes-manifests.yaml, buscar el deployment loadgenerator y cambiar los valores de las variables USERS y RATE a 1000 y 100 respectivamente.

Además debemos subir los recursos asignados al pod loadgenerator, para que pueda generar la carga necesaria. Cambiaremos los valores de requests y limits de CPU a 2000m y la memoria a 2048Mi.

Ejecutamos otra vez el siguiente comando para actualizar los valores:



```bash

kubectl apply -f ./release/kubernetes-manifests.yaml

```



A continuación monitorizaremos que deployment requiere más recursos para soportar la carga. Para ello usaremos el comando:



```bash

watch kubectl get pods

```

También podemos ver que peticiones genera el loadgenerator y cuales están siendo atendidas correctamente con el siguiente comando:



```bash

kubectl logs loadgenerator-<id-pod> --tail 20

```

Cuando observe que algún despliegue deja de estar disponible, ejecute el comando.



```bash

kubectl hpa <deployment-name> --min 1 --max 10

```

El comando anterior creará un autoescalador horizontal de pods (HPA) para el despliegue indicado, con un mínimo de 1 réplica y un máximo de 10 réplicas.

Para monitorizar el estado del HPA, podemos usar el comando:

```bash

kubectl get hpa

```

Si muestra unknown\_cpu, espere unos minutos hasta que se inicie el servidor de métricas y vuelva a ejecutar el comando.

¿Cuantos pods se han desplegado del servicio que se ha autoescalado?

**ENTREGA:** Se pide entregar el texto y los comandos que aparecen en la consola, así los comentarios pedidos a lo largo de la práctica.

**IMPORTANTE:**  Una vez acabada la práctica destruya las máquinas virtuales para evitar gastar recursos en ellas. Para ello utilice el interfaz de máquina virtuales (destruir todas) o el de clústeres de Kubernetes (un único cluster).


