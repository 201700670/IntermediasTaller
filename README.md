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
>- Construir el contenedor 
```console
docker build -t gcr.io/intermediafinal/node-mi-app  
```
>- Agregar el contenedor a la direccion
```console
docker push gcr.io/intermediafinal/node-mi-app 
```
>- Actualizar informacion
```console
docker pull gcr.io/intermediafinal/nodeapps
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

