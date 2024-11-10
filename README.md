# Guía de Despliegue - Aplicación de Dietas

Esta guía proporciona instrucciones detalladas para configurar y desplegar la aplicación de dietas en AWS y GCP. Incluye la configuración de una base de datos en RDS, el alojamiento del frontend en S3 y el despliegue del backend en un clúster de Kubernetes en Google Cloud Platform (GCP).


## Índice

1. [Configuración de RDS](#configuración-de-rds)
2. [Kubernetes en GCP](#kubernetes-en-gcp)
3. [Despliegue del Frontend en S3](#despliegue-del-frontend-en-s3)

---

## Configuración de RDS

### 1. Crear una instancia RDS

1. Inicia sesión en la consola de AWS.
2. Ve al servicio **RDS**.
3. Haz clic en **Create database**.
4. Selecciona las siguientes opciones:
   
   - **Database creation method**: Standard create
   - **Engine type**: PostgreSQL
   - **Version**: PostgreSQL 13.11-R2
   - **Template**: Free tier
   - **DB instance identifier**: `database-1`
   - **Master username**: `postgres`
   - **Credentials management**: Self managed
   - **Master password**: `cloud2024ufm`
   - **Confirm password**: `cloud2024ufm`
   - **DB instance class**: `db.t3.micro` (free tier)
   - **Storage type**: Dejar la opcion predeterminada de General Purpose SSD(gp3)
   - **Storage**: 20 GB (mínimo)
   - **Storage autoscaling**: Disabled
   - **VPC**: Default VPC
   - **Public access**: Yes
   - **VPC security group**: Create new
   - **Security group name**: `diet-db-sg`
   - **Database authentication**: Password authentication

### 2. Configurar el Grupo de Seguridad

1. Ve a **EC2 > Security Groups**.
2. Encuentra el grupo de seguridad creado para RDS.
3. Edita las reglas de entrada:
   - **Type**: PostgreSQL
   - **Port**: 5432
   - **Source**: `0.0.0.0/0` (o la IP específica de tu backend).

### 3. Obtener información de conexión

Guarda esta información para la configuración del backend:

- **Endpoint**
- **Puerto**: 5432
- **Nombre de usuario**: `postgres`
- **Contraseña**: `cloud2024ufm`
- **Nombre de la base de datos**: `database-1`

## Kubernetes en GCP

### Prerrequisitos

Instala el siguiente software:
- Google Cloud SDK
- kubectl
- Docker Desktop

### Instalación por Sistema Operativo

#### Windows
```powershell
# Descargar e instalar Google Cloud SDK
# https://cloud.google.com/sdk/docs/install

# Descargar e instalar Docker Desktop
# https://www.docker.com/products/docker-desktop
```

#### MacOS
Revisa si ya lo tiene instalado:
```bash
brew list | grep google-cloud-sdk
docker --version
```
Si aún no, instalarlo con Homebrew
```bash
# Instalar con Homebrew
brew install google-cloud-sdk
brew install --cask docker
```

#### Linux
```bash
# Seguir guía de instalación de Google Cloud SDK
# https://cloud.google.com/sdk/docs/install#linux

# Instalar Docker según tu distribución
# https://docs.docker.com/engine/install/
```

## Instalación Inicial
```bash
# Instalar kubectl a través de gcloud
gcloud components install kubectl

# Iniciar sesión en Google Cloud
gcloud auth login

# Configurar proyecto
gcloud config set project [ID-PROYECTO]
```

## Configuración del Proyecto

### 1. Crear una cuenta de Google Cloud
- Ve a [Google Cloud Console](https://console.cloud.google.com/)
- Si no tienes una cuenta, regístrate y configura la facturación

### 2. Crear un Nuevo Proyecto
1. En la consola de Google Cloud, haz clic en el selector de proyectos en la barra superior
2. Haz clic en "Nuevo Proyecto"
3. Ingresa:
   - Nombre del proyecto
   - Organización (opcional)
   - Ubicación (opcional)
4. Haz clic en "CREAR"

### 3. Obtener el ID del Proyecto
- El ID del proyecto se muestra en la consola
- También puedes verlo ejecutando:
   ```bash
   gcloud projects list
   ```

### 4. Configurar el Proyecto Activo
```bash
# Formato del comando
gcloud config set project [ID-PROYECTO]

# Ejemplo
gcloud config set project mi-proyecto-12345
```

### 5. Verificar la Configuración
```bash
# Verificar el proyecto actual
gcloud config get-value project
```

## Notas Importantes
- El ID del proyecto debe ser único en todo Google Cloud
- El ID del proyecto no se puede cambiar después de crearlo
- El ID puede ser diferente al nombre del proyecto
- Asegúrate de tener permisos suficientes en el proyecto


## Configuración del Cluster
```bash
# Crear cluster de Kubernetes
gcloud container clusters create mi-cluster \
    --zone us-central1-a \
    --num-nodes 2 \
    --machine-type e2-medium

# Configurar credenciales de la base de datos
kubectl create secret generic db-credentials \
    --from-literal=DB_USER=postgres \
    --from-literal=DB_PASSWORD=cloud2024ufm
```

## Despliegue de la Aplicación
- Asegurarse de colocar el endpoint del RDS que se creo en el configmap.yaml antes de correr lo siguiente.
```bash
# Aplicar configuraciones
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Verificación
```bash
# Verificar pods
kubectl get pods

# Verificar servicio y obtener IP externa
kubectl get services backend-service

# Verificar logs
kubectl logs -l app=backend
```

## Solución de Problemas

### Pods no inician
```bash
# Ver detalles del pod
kubectl describe pod [nombre-del-pod]
```

### Problemas de conexión a Base de Datos
1. Verificar secretos:
    ```bash
    kubectl get secrets
    ```
2. Verificar configuración:
    ```bash
    kubectl describe configmap backend-config
    ```

### Reiniciar Despliegue
```bash
kubectl rollout restart deployment backend-deployment
```

## Monitoreo
```bash
# Logs en tiempo real
kubectl logs -f [nombre-del-pod]

# Estado de los nodos
kubectl get nodes

# Estado de todos los recursos
kubectl get all
```

## Notas Importantes
- La imagen del backend está en: `josuer02/mi-backend:latest`
- Base de datos: AWS RDS
- Puerto del backend: 3001
- Los archivos YAML ya contienen todas las configuraciones necesarias

## Despliegue del Frontend en S3

### 1. Preparar el código para producción

1. Tras clonar el repositorio, en la carpeta `frontend` busca lo siguiente:

- app/page.tsx
- modifica la linea de `API_URL` e ingresa tu direccion de GCP

2. En tu proyecto local (dentro de la carpeta `frontend`), ejecuta:
   ```bash
   npm run build
   ```

### 2. Crear y configurar el bucket S3

1. Ve al servicio **S3** en la consola de AWS.
2. Haz clic en **Create bucket**.
3. Configura el bucket:
   - **Bucket name**: `diet-frontend-bucket` (o el nombre que prefieras).
   - **Region**: Selecciona la más cercana a tus usuarios.
   - **Block Public Access settings**: Desmarca **Block all public access**.
   - Haz clic en **Create bucket**.

### 3. Configurar el alojamiento web estático

1. Selecciona tu bucket.
2. Ve a la pestaña **Properties**.
3. Desplázate hasta **Static website hosting**.
4. Haz clic en **Edit**.
5. Configura:
   - Selecciona **Enable**
   - **Index document**: `index.html`
   - **Error document**: `index.html`
   - Guarda los cambios.

### 4. Configurar la política del bucket

1. Ve a la pestaña **Permissions**.
2. En **Bucket policy**, haz clic en **Edit**.
3. Pega la siguiente política (reemplaza `your-bucket-name` por el nombre que elegiste):
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::your-bucket-name/*"
       }
     ]
   }
   ```

### 5. Configurar CORS

1. En la pestaña **Permissions**.
2. Encuentra **Cross-origin resource sharing (CORS)**.
3. Haz clic en **Edit**.
4. Pega la siguiente configuración:
   ```json
   [
     {
       "AllowedHeaders": ["*"],
       "AllowedMethods": ["GET", "HEAD", "POST", "PUT", "DELETE"],
       "AllowedOrigins": ["*"],
       "ExposeHeaders": []
     }
   ]
   ```

### 6. Subir archivos al bucket

1. Ve a la pestaña **Objects**.
2. Haz clic en **Upload**.
3. Arrastra todos los archivos y carpetas de tu directorio `out`.
4. Mantén la estructura de carpetas intacta.
5. Haz clic en **Upload**.

### 7. Acceder al sitio web

1. Ve a la pestaña **Properties**.
2. Desplázate hasta **Static website hosting**.
3. Encontrarás la URL de tu sitio web (algo como `http://your-bucket-name.s3-website-region.amazonaws.com`).

### Verificación

1. Accede a la URL de tu sitio web.
2. Verifica que el diseño y los estilos se muestren correctamente.
3. Comprueba que puedas ver y crear dietas.
4. Verifica que la conexión con el backend funcione correctamente.
