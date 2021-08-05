# IntermediasTaller

### Requerimientos:
>- Tener una cuenta de Google Cloud
>- No es necesario tener docker 

## Comando para crear mi primer cluster
```phyton 
$ gcloud container clusters create "mi-first-clust" --num-nodes "2" --disk-size "32" --machine-type "g1-small" --zone "us-central1-c"
```
## Documentacion kubernetes
>-  https://kubernetes.io/es/docs/tasks/

#### Para instalar kubectl se necesita tener instalado de manera local chocolatey
>- https://docs.chocolatey.org/en-us/choco/setup#more-install-options
>- Windows:
```java
C:\WINDOWS\system32> @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```
>- luego instalar kubernetes cli
```python
$ choco install kubernetes-cli
```
>- verificar si instaló la version 
```java
$ kubectl version
```
>- Obtener credenciales de cluster, instalar SDK de google Cloud de manera local
```python
$ gcloud container clusters get-credentials mi-first-clust --zone us-central1-c
```
>- Crear un despliegue de kubernetes, crear un deployment conjunto de instrucciones que tiene un contenedor para que este en un pod 
> - - ngnix es una aplicacion
```java
$ kubectl create deployment hello-server --image=ngnix
```

## Despliegues configurados
```console
$ kubectl get deployment
```

## Mas informacion del despliegue
```console
$ kubectl describe deployment
```
## (Pods) Capa de extraccion 
```console
$ kubectl get pods
```
## Crear servicio para exponer deployment con LoadBalancer y expone directamente en un puerto de la aplicacion ngnix
```java
$ kubectl expose deployment hello-server --type=LoadBalancer --port=80
```
## Obtener los servicios (Recargar para obtener ip externo)
```console 
$ kubectl get services
```

