# Falabella K8s Candidate Skills Test 
Author: Sebastian García

## Objetivos
- Contruir una aplicación REST-based en Python
- Desplegar en Kubernetes a través del gestor de paquetes Helm
- Construir un Pipeline automatizado para desplegar la solución en Kubernetes.

## Introducción 
Se ha propuesto contruir una REST-base app en Python con el fin de poder desplegarla de manera automática con Kubernetes, utilizando un pipeline script donde se hiciera uso de Helm para la creación de un chart que me definiera un Deployment con las siguientes carácteristicas:
- 2 o más pods
- Definir un healtcheck que asegure la disponibilidad del servicio a través del path /health.
- Servicio de tipo ClusterIP
- Ingress que permita el tráfico hacia los pods.

## Instalación

Esta guía se ha construido bajo el sistema operativo Ubuntu18 y la solución de Kubernetes ha sido desplegada a través de GKE (Google Kubernetes Engine).
A su vez, se asume que cuenta con los siguientes pre-requisitos instalados, en caso de no ser así, se deja un link como sugerencia de guía de instalación.

### PRE REQUISITOS
- Kubernetes cluster solution (Minikube, local cluster, cloud cluster, etc).
- Docker -> https://docs.docker.com/engine/install/ubuntu/
- Docker hub
- Helm installed por el lado del cliente y servidor. --> https://v2.helm.sh/docs/using_helm/
- IDE favorito

## Desarrollo de la solución.

Dentro de la solución existe la carpeta 'hello-world-python' esta carpeta tiene la sgnte estructura:
+-- hello-world-python
+-- templates
|   +-- ingress.yaml
|   +-- microservice-deployment.yaml
+-- Char.yaml
+-- Values.yaml

La responsabilidad de esta carpeta es tener todos los archivos necesarios para la construcción del Chart que nos desplegara nuestra App.
El archivo microservice-deployment.yaml, define un objeto de tipo Deployment, el cual tiene configurado 2 replicas, la plantilla para los pods, los cuales tienen su healtcheck readinesProbe y livenesessprobe. A su vez, en este archivo se define el servicio tipo ClusterIP en el cual van a vivir los PODS.

ingress.yaml define la creación del ingress y solamente tiene un path por el cual escuchar peticiones.

Values.yaml tiene todas las variables de configuración para el deployment.  

Dentro de la solución existe la carpeta 'hello-world-python-code', esta carpeta contiene el código de la Aplicación, el Dockerfile para construir la imagen de la aplicación y los respectivos requierements, para la inslación de dependencias de python.

---- Continuará -----



