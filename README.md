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
```
+--_hello-world-python
+--_templates
|   +--_ingress.yaml
|   +--_microservice-deployment.yaml
+--_Char.yaml
+--_Values.yaml  
````

La responsabilidad de esta carpeta es tener todos los archivos necesarios para la construcción del Chart que nos desplegara nuestra App.
El archivo microservice-deployment.yaml, define un objeto de tipo Deployment, el cual tiene configurado 2 replicas, la plantilla para los pods, los cuales tienen su healtcheck readinesProbe y livenesessprobe. A su vez, en este archivo se define el servicio tipo ClusterIP en el cual van a vivir los PODS.

**ingress.yaml** define la creación del ingress y solamente tiene un path por el cual escuchar peticiones.

**Values.yaml** tiene todos los valores de las variables de configuración para el deployment, service e ingress.  

Dentro de la solución existe la carpeta `hello-world-python-code`, esta carpeta contiene el código de la Aplicación, el Dockerfile para construir la imagen de la aplicación y los respectivos requierements, para la instalación de dependencias de python.
```
hello-world-python-code/
├── Dockerfile
├── requirements_dev.txt
├── requirements.txt
├── src
│   └── launch.py
└── tests
    ├── __init__.py
    └── test_launch.py

2 directories, 7 files
```

El **Dockerfile** copia el código de la aplicación dentro del contenedor, instala solo los requerimientos del archivo **requirements.txt** ya que este tiene solo las dependecias necesarias para correr la aplicación, crea un ENV para que FLask corra en modo producción, expone el puerto por el cual corre la app y ejecuta la aplicación. El Dockerfile solo debería de compilarse para una aplicación que esté lista para producción.     

⋅⋅⋅**requirementes_dev.txt** => tiene las dependencias necesarias para compilar el código en modo desarrollador, de tal manera que tenga en cuenta la instalación de los módulos para pruebas.  

⋅⋅⋅**src** -> contiene la aplicación, esta aplicación tiene solo dos endpoint configurados: "/" y "/health"  
⋅⋅⋅**test** -> Carpeta con el código para correr las pruebas unitarias del código.  

Por otro lado, dentro de la raíz de este proyecto hay dos archivos muy importantes.
```
k8s_test
├── config.txt
├── pipeline.sh
└── README.md
```

- **config.txt** => este archivo contiene los parametros de configuración necesaria para hacer un proceso de despliegue automático, más reutilizable.
⋅⋅⋅la variable `launcher_version` hace referencia a la versión que le quieres taguear a tu pipeline.  
⋅⋅⋅la variable `directory_logs` hace referencia a la ruta del directorio donde quieres que se generen los logs de ejecución  
⋅⋅⋅la variable `docker_hub_repo`=> hace referencia al nombre del repositorio en docker hub donde se va almacenar la imagen.  
⋅⋅⋅la variable `tag_image` = hace referencia al tag de versionamiento que le quieres asignar a la contrucción de la imagen.  
⋅⋅⋅la variable `tag_name` = hace referencia al nombre que se le pondrá a la docker image.  

- **pipeline.sh** => Este archivo es un ejecutable escrito en bash el cual tiene el siguiente proceso de ejecución:
⋅⋅⋅Detalles de inicio
⋅⋅⋅**make_build_and_test()** => en este paso, lo que se hace es utilizar docker para construir una imagen temporal de la app en modo desarrollo y se corren las pruebas unitarias con pytest. Si la ocurrencia de la palabra FAILED en los logs de ejecución es ms grande que cero, entonces quiere decir que las pruebas no pasaron satisfactoriamente y por lo tanto, el pipeline aborta.  
⋅⋅⋅**package()** => Una vez las pruebas pasaron, se procede a empaquetar la aplicación, que en este caso es un proceso de contruir la imagen desde nuestro Dockerfile que ya tenemos listo para versiones de producción, nos logueamos (docker login) a docker, y hacemos PUSH de la imagen a nuestro repositorio.  
⋅⋅⋅**deploy()** => Una vez la aplicación ya se ha empaquetado, se procede a desplegar la imagen en Kubernetes. La función primero extrae del archivo values.yaml los el valor de la anterior imagen, seguidamente construye el nombre de la siguiente imagen y reemplaza en el archivo values.yaml el nombre de la nueva imagen. 
Hace una busqueda en la lista de los chart desplegados con helm, para determinar si se encuentra ya desplegado el chart, si se encuentra desplegado entonces hace un upgrade, de lo contrario, procede a instalar un nuevo release del chart.  
⋅⋅⋅**smoke_test()** => Se deja declarado (TODO) para posteriores versiones.

### Ejecución del proyecto.

Nota: Se mostrará el proceso de ejecución para desplegar en GKE (Google Kubernetes Engine) de igual modo, la solución es suficiente clean para poderse ejecutar en cualquier tipo de Cluster de Kubernetes que tú desees.

1. Hacemos un clone del repo ```git clone https://github.com/ikermatias/k8s_test/```
2. Ingresamos al directorio ```cd k8s_test```
3. Garantizamos privilegios de ejecución al archivo pipeline.sh ```sudo chmod +x pipeline.sh```
3. Garantizamos que estemos logueados a nuestro cluster de Kubernetes, ejemplo GKE ```gcloud container clusters get-credentials <name of cluster> --zone <zone> --project <project_id>``` A su vez, recordemos que debemos tener instalado HELM por el lado del cliente y del cluster.
4. Ejecutamos el archivo pipeline.sh ```./pipeline```
5. Listo ! solo nos queda esperar la creación del ingress para poder tener nuestra EXTERNAL_IP y empezar a consumir nuestro servicio.
```kubectl get ingress```
NOTA: Recuerde que Nginx ingress controller en GKE trabaja con CLusterIP, nativos ingress controller no trabajan con ClusterIP, por lo tanto su servicio para pods debera de ser NodePort. Tenga en cuenta esto si va a desplegar la solución en un ambiente diferente a GKE, simplemente en el microservice-deployment.yaml cambie el tipo de servicio a NodePort. ejemplo:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```


## Oportunidades de mejora.

¿Que faltaría para poder este servicio ser desplegado en producción?
¿Qué se le agregara con ms tiempo?
