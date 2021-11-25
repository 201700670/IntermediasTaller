# Taller Docker básico

### Requerimientos:
>- Tener una cuenta de Google Cloud
>- No es necesario tener docker por el momento

## Documentacion kubernetes
>-  https://kubernetes.io/es/docs/tasks/

#### Para instalar kubectl se necesita tener instalado de manera local chocolatey
>- https://docs.chocolatey.org/en-us/choco/setup#more-install-options
>- Windows:
```java
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```
>- luego instalar kubernetes cli
```python
choco install kubernetes-cli
```
>- verificar si instaló la version 
```java
kubectl version
```

## Comando para crear mi primer cluster
```phyton 
gcloud container clusters create "cluster-first" --num-nodes "2" --disk-size "32" --machine-type "g1-small" --zone "us-central1-c"
```
## Agregar credenciales
```java
gcloud container clusters get-credentials cluster-first --zone us-central1-c
```
### Archivos para Pods que son uno o mas contenedores de docker
>- Crearemos archivos
#### informacion de prueba pods correctos 
>- http://www.yamllint.com/

> - - nano app.yaml
```console

apiVersion: apps/v1             # Versión de kubernetes
kind: Deployment                # Tipo de Objeto 
metadata:                       # Información adicional
  name: node-deployment         # Nombre del despliegue
  labels:                       # Etiquetas con clave valor, sirven para identificar       
    app: node-app               # Etiqueta para el despliegue
spec:                           # Especificaciones del despliegue
  replicas: 1                   # Número de replicas que tendrá este pod
  selector:                     # Es la forma primitiva de hacer referencia, es como un id interno
    matchLabels:                # 
      app: node-app             # Etiqueta para que otros objetos de kubernetes lo encuentren
  template:                     # Es la configuración final del pod 
    metadata:                   # Tiene su propia metadata para la plantilla o pod
      labels:                   # 
        app: node-app           # Etiqueta del contenedor
    spec:                       # Tiene sus especificaciones
      containers:               # Información del contenedors o mas si el pod tiene muchos
      - name: node-app          # Nombre del contenedor
        image: gcr.io/intermediaprueba/node-mi-app      # url de imagen, docker hub defautl
        ports:                  # puertos
        - containerPort: 8080   # puerto 80
```

> - - basic-ingress.yaml
```console
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: basic-ingress
spec:
  backend:
    serviceName: node-app
    servicePort: 8080
```

> - - service-node.yaml
```console
apiVersion: v1
kind: Service
metadata:
  name: node-app
  namespace: default
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: node-app
  type: NodePort
```

### CREACION DE CONTENEDOR Y ELEMENTOS 
#### Creacion de carpetas y archivos 
>- Carpeta nodoapp
```console
mkdir nodeapp
nano Dockerfile
nano index.js
nano package.json
```
>- nano Dockerfile
```Dockerfile
FROM node:12-buster-slim
WORKDIR /usr/src/app
COPY . .
RUN npm install
ENTRYPOINT node index.js
```
>- nano index.js
```javascript
const express= require('express');
const app= express();
app.get('/',(req,res)=>{
	res.send('Hello World');
});

app.listen(8080, '0.0.0.0');
console.log("running on http://0.0.0.0:8080");
```
>- nano package.json
```json
{
	"name": "docker_web_app",
	"version": "1.0.0",
	"description": "Node.js on Docker",
	"author": "First Last <first.last@example.com>",
	"main": "server.js",
	"scripts": {
		"start": "node server.js"
	},
	"dependencies": {
		"express": "^4.17.1"
	}
}
```

## Comandos DOCKER
>- Construir el contenedor (gcr.io/[nombre_proyecto]/[nombre_contenedo_repo])
```console
docker build -t gcr.io/intermediafinal/node-mi-app .
```
>- Agregar el contenedor a la direccion (gcr.io/[nombre_proyecto]/[nombre_contenedor_repo])
```console
docker push gcr.io/intermediafinal/node-mi-app 
```
>- Actualizar informacion (gcr.io/[nombre_proyecto]/[nombre_contenedor_repo])
```console
docker pull gcr.io/intermediafinal/node-mi-app
```
>- Crear un despliegue de kubernetes, crear un deployment conjunto de instrucciones que tiene un contenedor para que este en un pod  
```console
kubectl create deployment node-mi-app-deploy --image gcr.io/intermediafinal/node-mi-app 
```
>- Crear servicio para exponer deployment con LoadBalancer y expone directamente en un puerto de la aplicacion 
```console
kubectl expose deployment node-mi-app-deploy --type=LoadBalancer --port 8080
```
## Despliegues configurados
```console
kubectl get deployment
```

## Despliegues configurados
```console
kubectl get deployment
```

## Mas informacion del despliegue
```console
kubectl describe deployment
```
## (Pods) Capa de extraccion 
```console
kubectl get pods
```
## Obtener los servicios (Recargar para obtener ip externo)
```console 
kubectl get services
```
# Informacion de Pods
>- https://kubernetes.io/es/docs/concepts/workloads/pods/pod/

## Ejecucion de archivos yaml de Pods 
```console
kubectl apply -f app.yaml
```
```console
kubectl apply -f service-node.yaml
```
```console
kubectl apply -f basic-ingress.yaml
```
# MONITOREO
## Grafana y Prometheus

#### Prometheus Operator for Kubernetes

Un operador de Kubernetes es un método para empaquetar, implementar y gestionar un aplicación de esta plataforma, que se implementa ahí mismo y se administra con la interfaz de programación de aplicaciones (API) de kubernetes y la herramienta kubectl

##### Arquitectura de Prometheus 
Prometheus se compone de múltiples componentes si bien estos son los principales para el caso de uso definido previamente:

>- Prometheus server: Es el componente encargado de recolectar y almacenar las métricas de los aplicativos en una time series database.
>- Service discovery: Prometheus dispone de conectores con los principales Service Discovery del mercado pudiendo auto descubrir y desechar aplicaciones de forma automática en tiempo real, algo fundamental cuando se trabaja con contenedores que cambian de IP constantemente.
>- Client libraries: Son las librerías encargadas de exponer las métricas internas de la aplicación a monitorizar en formato Prometheus (CPU, Memoria, Threads, GC…), para que puedan ser recolectadas por el Prometheus server.
>- Alert manager: Es el componente encargado de gestionar las alertas enviadas por el Prometheus server. Es decir, agrupa y notifica las alarmas por los distintos medios definidos como correo electrónico, PagerDuty u OpsGenie entre otros. También se encarga de silenciar e inhibir las alertas.
>>>
![image](https://user-images.githubusercontent.com/36779113/143400826-eac60b25-1602-418b-8267-23789ae81ddb.png)

##### Implementacion
> 1. Descargar el repositorio
```git
git clone https://github.com/prometheus-operator/kube-prometheus
```
> 2. Ingresamos a la carpeta del repositorio
```console
cd kube-prometheus
```
> Aplicamos la pila de kube-prometheus 
The previous steps (compilation) has created a bunch of manifest files in the manifest/ folder. Now simply use kubectl to install Prometheus and Grafana as per your configuration:

```console
# Update the namespace and CRDs, and then wait for them to be available before creating the remaining resources
$ kubectl apply -f manifests/setup
$ kubectl apply -f manifests/
```

> Obtener todos los pods 
```console
kubectl get all -n monitoring
```
![image](https://user-images.githubusercontent.com/36779113/143404562-efa0ce09-cda7-4c81-a072-b7d1a43fbacc.png)
![image](https://user-images.githubusercontent.com/36779113/143404766-f732374c-9a70-4b35-9682-5812761b1567.png)

> Exponer en puerto 9090 prometheus
![image](https://user-images.githubusercontent.com/36779113/143405235-1cb5dc93-4c3d-45be-821b-d922796e4c42.png)

> Exponer en puerto 3000 graphana
![image](https://user-images.githubusercontent.com/36779113/143405332-bcb386fb-152c-4608-b170-a0e5a3be295c.png)
![image](https://user-images.githubusercontent.com/36779113/143405499-e75908e9-2358-4573-85ac-f724f8245ad6.png)
