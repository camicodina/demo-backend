# Deploy en Openshift con Tekton (IBM CP4Apps)

## Primeros pasos: creación de aplicación con Appsody

```bash
$appsody repo add kabanero https://github.com/kabanero-io/kabanero-stack-hub/releases/download/0.6.3/kabanero-stack-hub-index.yaml
$appsody repo list
$appsody repo set-default kabanero
$appsody list kabanero
```

En este caso estaremos usando el template java-spring-boot2 default

```bash
$appsody init kabanero/java-spring-boot2
```

Test:
```bash
$appsody run
```

## Codewind 

Podemos agregar el proyecto al Workspace de Codewind para testeos y observar las métricas

## Cambios en código

```html
<dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.8</version>
    </dependency>
```

## Pre-req de Openshift

Usando las credenciales de Openshift entraremos a la cuenta de Openshift usada para el deploy del CP4Apps y crearemos un nuevo proyecto.

```bash
$oc new-project <yournamespace> 
$oc project <yournamespace> 
```
Ahora debemos hacer que nuestra aplicación que usa kabanero acceda a los recursos de kabanero.
Gracias a la instalación de CP4Apps ya están disponibles en un proyecto particular en Openshift (kabanero).

```bash
$oc get kabaneros -n kabanero 
$oc edit kabanero kabanero -n kabanero 
(:x para guardar cambios)
```

Este comando nos dejará editar el yaml que configura kabanero.
Buscaremos la label "spec" para agregar el nuevo proyecto a los target namespaces 

```yaml
spec:
  targetNamespaces:
    - <yournamespace>
```

## Crear archivo de deployment

Appsody nos generará un app-deploy.yaml en la carpeta de nuestro proyecto.

```bash
$appsody deploy --generate-only 
```

Por defecto, la aplicación es deployeada en el namespace 'kabanero'.
Si queremos deployear en nuestro nuevo namespace, podemos especificarlo abajo de la metadata.

```yaml
apiVersion: appsody.dev/v1beta1
kind: AppsodyApplication
metadata:
  name: quote-backend
  namespace: <yournamespace>
spec:
  # Add fields here
```

## Crear repositorio de Github

Crearemos un proyecto de github y, volviendo a nuestro proyecto, los vincularemos.

```bash
$git init
$git add .
$git commit -m "initial commit" 
$git remote add origin $GITHUB_REPOSITORY_URL
$git push -u origin master
```

También createmos un access token para poder generar un Webhook en Tekton:
Github Settings > Developer Settings > Personal Access Tokens > Generate New Token (permiso 'admin:repo_hook')

## Jugando con Tekton

Accederemos al dashboard de Tekton desde el menu del CP4Apps (en la solapa Pipelines) usando las credenciales de Openshift.
En Tekton, en el margen izquierdo, podremos ver la opción de crear Webhooks, agregaremos la información:

```bash
Name - <Name for webhook>
Repository URL - <Your github repository URL>
Access Token - <Your access token>
Namespace - kabanero
Pipeline - java-spring-boot2-build-deploy-pl
Service account - kabanero-operator
Docker Registry - image-registry.openshift-image-registry.svc:5000/<your_project>
```

y le daremos a crear.

En esta demo estaremos usando el Docker Registry por defecto del cluster de Openshift. Podríamos configurar un registro externo.

Haciendo un pequeño cambio en el código como el título y pusheando los cambios a Github podremos ver cómo el Webhook crea un nuevo deployment.

¡Desde la GUI de Tektok y Openshift observaremos la magia!



 