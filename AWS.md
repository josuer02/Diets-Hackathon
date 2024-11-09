# Guía de Despliegue - Aplicación de Dietas

Esta guía proporciona instrucciones detalladas para configurar y desplegar la aplicación de dietas en AWS, específicamente para configurar RDS (base de datos) y hostear el frontend en S3.

## Índice

1. [Configuración de RDS](#configuración-de-rds)
2. [Despliegue del Frontend en S3](#despliegue-del-frontend-en-s3)

## Configuración de RDS

### 1. Crear una instancia RDS

1. Inicia sesión en la consola de AWS.
2. Ve al servicio **RDS**.
3. Haz clic en **Create database**.
4. Selecciona las siguientes opciones:
   - **Engine type**: PostgreSQL
   - **Version**: PostgreSQL 13.11-R2
   - **Template**: Free tier
   - **DB instance identifier**: `database-1`
   - **Master username**: `postgres`
   - **Master password**: `cloud2024ufm`
   - **Confirm password**: `cloud2024ufm`
   - **DB instance class**: `db.t3.micro` (free tier)
   - **Storage**: 20 GB (mínimo)
   - **Storage autoscaling**: Disabled
   - **Availability & durability**: Single DB instance
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
