# PRÁCTICA 6: CREACIÓN DE UN CLÚSTER DE KUBERNETES Y DESPLIEGUE DE APLICACIONES

## Descripción

En esta práctica, el alumno aprenderá a desplegar un clúster local de Kubernetes mediante **Minikube** y a desplegar diversas aplicaciones sobre él. También se realizarán tareas de exposición de servicios, escalado manual, monitorización de recursos y autoescalado horizontal de Pods (HPA).

## Objetivos

* Afianzar los conocimientos adquiridos en la gestión de contenedores y orquestación de aplicaciones.
* Aprender a realizar tareas de desarrollador y de administrador en entornos Kubernetes.
* Comprender los mecanismos de despliegue, descubrimiento de servicios, balanceo de carga y escalado automático en microservicios.

## Requisitos previos

Para realizar esta práctica en los equipos del laboratorio se requiere disponer de **Docker** (utilizado en la práctica anterior), **`kubectl`** y **`minikube`**. Antes de comenzar, compruebe que las herramientas están disponibles ejecutando:

```bash
docker --version
minikube version
kubectl version --client
```

> **Nota sobre recursos:** La segunda parte de la práctica despliega una aplicación completa de microservicios (~11 Pods). Se recomienda arrancar Minikube con al menos **4 CPUs y 4096 MB de RAM** (o `6144` MB si el equipo dispone de 16 GB o más).

---

## Actividades a desarrollar

La práctica se divide en dos partes: en primer lugar el despliegue y escalado manual de una aplicación sencilla, y a continuación el despliegue, generación de carga y autoescalado de una arquitectura basada en microservicios.

A lo largo de la práctica se deberá elaborar una **memoria** en la que se reflejen los pasos seguidos, las capturas/salidas relevantes de consola y las respuestas a todas las preguntas y reflexiones indicadas en negrita.

---

### Parte 1: Creación del clúster y despliegue de una aplicación sencilla

1. **Creación del clúster local:** Inicie un clúster de Kubernetes con Minikube utilizando Docker como *driver*:

   ```bash
   minikube start --driver=docker --cpus=4 --memory=4096
   ```

   Espere unos instantes hasta que el clúster esté listo. Verifique su estado y compruebe qué ha ocurrido en Docker ejecutando:

   ```bash
   minikube status
   kubectl get nodes
   docker ps
   ```

   * **Indique qué diferencias cree que existen entre el uso de `docker-compose` (práctica 5) y la gestión de contenedores con Kubernetes realizada en esta práctica.**
   * **Realice una captura de `kubectl get nodes` y `docker ps`, e indique de cuántos nodos está formado el clúster y cómo está ejecutando Minikube dicho nodo en su equipo.**

2. **Panel de control del clúster (opcional):** De forma análoga a la consola web de proveedores Cloud (como Google Kubernetes Engine), Minikube permite abrir un panel gráfico de administración. Si desea explorarlo, ejecute el siguiente comando en una **terminal aparte**:

   ```bash
   minikube dashboard
   ```

3. **Despliegue de una aplicación sencilla:** Ahora que disponemos de un clúster donde desplegar Pods (formados por contenedores), crearemos un despliegue utilizando la imagen oficial de pruebas de Kubernetes (`agnhost`), arrancando su servidor HTTP en el puerto `8080`:

   ```bash
   kubectl create deployment hello-node \
     --image=registry.k8s.io/e2e-test-images/agnhost:2.53 \
     -- /agnhost netexec --http-port=8080
   ```

   Podemos inspeccionar el estado del despliegue y la configuración del clúster con los siguientes comandos:

   ```bash
   kubectl get deployments
   kubectl get pods
   kubectl get events
   kubectl config view
   ```

   Espere hasta que el Pod pase a estado `Running`.
   * **Indique qué información nos muestra cada uno de los cuatro comandos anteriores.**

4. **Exposición del servicio:** Por defecto, los Pods solo son accesibles desde la red interna del clúster. Para exponer el despliegue al exterior, crearemos un *Service* de tipo `LoadBalancer` en el puerto `8080`:

   ```bash
   kubectl expose deployment hello-node --type=LoadBalancer --port=8080
   kubectl get services
   ```

   Observará que en Minikube el campo `EXTERNAL-IP` aparece como `<pending>`, ya que no existe un balanceador de carga de un proveedor Cloud asignando una IP pública real. Para obtener una URL de acceso local al servicio, ejecute:

   ```bash
   minikube service hello-node --url
   ```

   *(Mantenga esa terminal abierta si el comando queda en primer plano, o abra la URL devuelta directamente en su navegador o con `curl`).*

   Pruebe también a añadir `/hostname` al final de la URL (por ejemplo, `http://127.0.0.1:<puerto>/hostname`) para ver qué Pod está respondiendo a la petición.
   * **Indique en la memoria cuál es el punto de acceso obtenido y qué respuesta devuelve el servidor.**

5. **Escalado manual:** Escale el despliegue a 4 réplicas y compruebe el resultado:

   ```bash
   kubectl scale deployments/hello-node --replicas=4
   kubectl get pods
   kubectl get deployments
   ```

   Realice varias peticiones seguidas a la ruta `/hostname` de su servicio (con el navegador o mediante `curl`).
   * **Indique cuántos contenedores están ejecutando el servidor, qué observa al repetir las peticiones a `/hostname` y realice una comparación con el escenario planteado en la práctica creativa 1.**

6. **Limpieza de la Parte 1:** Antes de desplegar la aplicación de microservicios, elimine el servicio y el despliegue de prueba para liberar recursos:

   ```bash
   kubectl delete service hello-node
   kubectl delete deployment hello-node
   ```

---

### Parte 2: Despliegue de una aplicación basada en microservicios

Como ejemplo utilizaremos una tienda *online* (*Online Boutique*) formada por múltiples microservicios independientes. Puede consultar su arquitectura y descripción en:
<https://github.com/GoogleCloudPlatform/microservices-demo>

1. **Descarga y despliegue de la aplicación:** Clone el repositorio y aplique los manifiestos de Kubernetes:

   ```bash
   git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
   cd microservices-demo
   kubectl apply -f ./release/kubernetes-manifests.yaml
   ```

   Espere a que todos los Pods se descarguen y pasen a estado `Running` (puede tardar un par de minutos). Verifique el estado con:

   ```bash
   kubectl get pods
   kubectl get services
   ```

   * **Indique cuántos Services y cuántos Pods se han configurado y arrancado.**

2. **Acceso a la tienda *online*:** Obtenga la URL de acceso al servicio `frontend-external` ejecutando:

   ```bash
   minikube service frontend-external --url
   ```

   > **Alternativa:** Si lo prefiere, también puede redirigir el puerto localmente con `kubectl port-forward service/frontend-external 8080:80` y acceder desde su navegador a `http://localhost:8080`.

   Acceda a la tienda *online* desde su navegador web y compruebe que puede navegar por el catálogo y añadir productos al carrito.
   * **Indique cuántos nodos forman el clúster y cuántos Pods tenemos arrancados en este momento.**

3. **Activación del servidor de métricas:** Para demostrar la capacidad de escalado automático de Kubernetes, utilizaremos el autoescalador horizontal de Pods (**HPA**, *Horizontal Pod Autoscaler*). El HPA necesita un servidor de métricas (*Metrics Server*) que recopile el consumo de CPU y memoria de los Pods. A diferencia de entornos gestionados como GKE donde viene preinstalado, en Minikube debemos habilitarlo manualmente:

   ```bash
   minikube addons enable metrics-server
   ```

   Espere aproximadamente un minuto hasta que el servidor de métricas comience a recopilar datos y compruebe que funciona correctamente con:

   ```bash
   kubectl top nodes
   kubectl top pods
   ```

4. **Generación de carga sobre la tienda:** La aplicación incluye un generador de tráfico (el despliegue `loadgenerator`) basado en un *script* de *Locust* que simula usuarios accediendo a la tienda. Para aumentar la carga del sistema:

   * Abra el fichero `release/kubernetes-manifests.yaml` y localice el `Deployment` llamado `loadgenerator`.
   * Cambie los valores de las variables de entorno `USERS` y `RATE` a `"150"` y `"20"` respectivamente (valores ajustados para un clúster local; si el equipo es muy potente y apenas sube la CPU, puede incrementarlos a `"300"` y `"40"`).
   * Ajuste los recursos asignados al contenedor `main` del Pod `loadgenerator` para que pueda generar dicha carga sin quedar retenido en estado `Pending`:
     * `requests`: `cpu: 500m`, `memory: 512Mi`
     * `limits`: `cpu: 1000m`, `memory: 1024Mi`

   Aplique los cambios ejecutando de nuevo:

   ```bash
   kubectl apply -f ./release/kubernetes-manifests.yaml
   ```

5. **Monitorización del sistema bajo carga:** En distintas terminales (o alternando comandos), supervise el consumo de recursos, el estado de los Pods y las peticiones generadas por `loadgenerator`:


   ```bash
   watch kubectl top pods
   ```

   ```bash
   watch kubectl get pods
   ```

   ```bash
   kubectl logs deploy/loadgenerator --tail=20
   ```

   * **Observe qué microservicio(s) presentan mayor consumo de CPU o empiezan a sufrir errores/saturación bajo la carga generada (habitualmente `frontend` o `productcatalogservice`). Indíquelo en la memoria.**

6. **Configuración del Autoescalador Horizontal de Pods (HPA):** Cree un HPA sobre el despliegue que haya identificado con mayor saturación de CPU (por ejemplo, `frontend`), estableciendo un objetivo de uso medio de CPU del 50%, un mínimo de 1 réplica y un máximo de 10:

   ```bash
   kubectl autoscale deployment <deployment-name> --cpu-percent=50 --min=1 --max=10
   ```

   Monitorice la evolución del HPA y de los Pods creados con:

   ```bash
   watch kubectl get hpa
   ```

   *(Si inicialmente la columna `TARGETS` muestra `<unknown>/50%`, espere entre 30 y 60 segundos a que el servidor de métricas actualice las lecturas).*

   * **¿Cuántos Pods se han llegado a desplegar del servicio autoescalado? Adjunte la salida de `kubectl get hpa` y `kubectl get pods` y explique qué ocurre con el porcentaje de CPU a medida que aumentan las réplicas.**

---

## Entrega

Se deberá entregar una **memoria** que incluya:
* Los comandos ejecutados y las salidas o capturas de consola más relevantes de cada apartado.
* Las respuestas razonadas a todas las preguntas y reflexiones planteadas en negrita a lo largo del enunciado.

---

## Limpieza al finalizar la práctica

**IMPORTANTE:** Una vez finalizada la práctica, detenga y elimine el clúster de Minikube para liberar la memoria RAM, la CPU y el espacio en disco del equipo del laboratorio:

```bash
minikube delete
```
