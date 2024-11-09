# Guía de Implementación - Proyecto Backend con Kubernetes

## Tabla de Contenidos
- [Prerrequisitos](#prerrequisitos)
- [Instalación Inicial](#instalación-inicial)
- [Configuración del Proyecto](#configuracion-del-proyecto)
- [Configuración del Cluster](#configuración-del-cluster)
- [Despliegue de la Aplicación](#despliegue-de-la-aplicación)
- [Verificación](#verificación)
- [Solución de Problemas](#solución-de-problemas)
- [Monitoreo](#monitoreo)
- [Notas Importantes](#notas-importantes)

## Prerrequisitos

### Software Necesario
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
