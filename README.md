# Monitoreo con Datadog y despliegue de imágenes en AWS

Este proyecto demuestra cómo:

- Crear una instancia EC2 con Terraform
- Instalar el agente de Datadog
- Subir imágenes Docker a ECR
- Ejecutar contenedores y visualizar métricas y vulnerabilidades en Datadog

---

## 🔧 Requisitos

- Cuenta de AWS
- Cuenta en Datadog (región EU)
- Terraform instalado
- Docker y AWS CLI configurados

---

## 🚀 Infraestructura con Terraform

```bash
terraform init
terraform apply
```

Esto crea una instancia EC2 (tipo `spot`) con:

- Acceso SSH, HTTP, HTTPS, SonarQube UI
- Docker y Docker Compose instalados
- Posibilidad de instalar el agente de Datadog vía `user_data`

---

## 📦 Instalación del agente de Datadog

En la instancia EC2, ejecutar:

```bash
DD_API_KEY=<tu_api_key> DD_SITE="datadoghq.eu" bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

Verificar con:

```bash
sudo datadog-agent status
```

---

## ⚙️ Configuración del agente (`/etc/datadog-agent/datadog.yaml`)

Agregar o modificar las siguientes secciones:

```yaml
container_image:
  enabled: true

sbom:
  enabled: true
  container_image:
    enabled: true
  host:
    enabled: true
```

Reiniciar el agente:

```bash
sudo systemctl restart datadog-agent
```

---

## ☁️ Integración de AWS con Datadog

Para sincronizar imágenes de ECR sin ejecutarlas:

1. En Datadog, ir a Integrations > Amazon Web Services
2. Seguir el asistente y desplegar la plantilla de CloudFormation sugerida
3. Asegurarse de activar las opciones:
   - "ECR"
   - "Cloud Security Posture Management" (si se desea ver vulnerabilidades)

---

## 🐳 Subir una imagen a ECR

1. Crear el repositorio:

```bash
aws ecr create-repository --repository-name mi-imagen-ecr --region eu-west-1
```

2. Login a ECR:

```bash
aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin <account>.dkr.ecr.eu-west-1.amazonaws.com
```

3. Build y push:

```bash
docker build -t mi-imagen-ecr .
docker tag mi-imagen-ecr:latest <account>.dkr.ecr.eu-west-1.amazonaws.com/mi-imagen-ecr:latest
docker push <account>.dkr.ecr.eu-west-1.amazonaws.com/mi-imagen-ecr:latest
```

---
## 🔍 Ver métricas y vulnerabilidades en Datadog

1. Ir a [Infrastructure > Containers](https://app.datadoghq.eu/infrastructure)
2. Para vulnerabilidades:
   - [Cloud Security > Container Images](https://app.datadoghq.eu/cws/inventory/container-images)
3. Usar filtros como:
   - `container_name:contenedor-prueba`
   - `image.name:<ecr-uri>`

---

# Integración de AWS con Datadog (sin agente)

Este documento describe cómo integrar tu cuenta de AWS con Datadog utilizando la integración nativa basada en API, sin necesidad de instalar un agente. Esto permite a Datadog recolectar métricas, logs (opcional) y otros datos directamente desde los servicios gestionados por AWS.

---

## 🛠️ Requisitos

- Cuenta de AWS con permisos para crear roles IAM
- Cuenta de Datadog con permisos para administrar integraciones
- Plan de Datadog con soporte para integraciones en la nube

---

## 🔗 Paso 1: Instalar la integración de AWS en Datadog

1. Inicia sesión en [Datadog](https://app.datadoghq.com/).
2. Ve a **Integrations > Integrations**.
3. Busca **Amazon Web Services** y haz clic en **Install** (si aún no está instalada).

---

## 🔐 Paso 2: Conectar una cuenta de AWS

### Opción A — 🔄 Crear el rol IAM automáticamente desde Datadog (recomendado)

1. En la configuración de la integración de AWS en Datadog, haz clic en **"Set up"**.
2. Elige **"Create Role Manually"** y luego selecciona **"Create Role with AWS Console"**.
3. Datadog abrirá una página en la consola de AWS con el formulario prellenado:
   - ID de cuenta de Datadog
   - External ID
   - Política mínima recomendada
4. Revisa y crea el rol IAM.
5. Copia el **ARN del rol** y pégalo en la configuración de la integración de AWS en Datadog.

### Opción B — 🧱 Crear el rol IAM manualmente desde cero

1. Accede a la [Consola de IAM de AWS](https://console.aws.amazon.com/iam/).
2. Ve a **Roles > Create Role**.
3. Selecciona **Another AWS Account**.
4. Ingresa el ID de cuenta de Datadog:
   - Para `us1`: `464622532012`
   - Otros sitios: ver [cuentas por región](https://docs.datadoghq.com/integrations/amazon_web_services/?tab=roledelegation)
5. Activa **Require external ID** e ingresa el **External ID** que aparece en Datadog.
6. Adjunta la política `ReadOnlyAccess` o una política personalizada proporcionada por Datadog.
7. Asigna un nombre al rol (por ejemplo, `DatadogIntegrationRole`) y crea el rol.
8. Copia el **ARN del rol** y agrégalo en la integración de Datadog.

---

## 🌐 Paso 3: Configurar regiones y servicios

1. En la interfaz de Datadog, especifica:
   - Las **cuentas de AWS** a monitorear (puedes agregar múltiples)
   - Las **regiones de AWS** donde tienes servicios
   - Los **servicios que deseas monitorear** (EC2, RDS, Lambda, etc.)

2. Guarda la configuración.

---

## 📊 ¿Qué podrás monitorear?

Datadog recolectará automáticamente datos de servicios como:

- EC2, EBS, Auto Scaling
- RDS, DynamoDB, Redshift
- Lambda, API Gateway
- ELB, CloudFront, VPC
- S3, CloudWatch
- y más

---

## 📁 Logs (opcional)

Para enviar logs desde CloudWatch a Datadog sin instalar agentes:

1. Crea una función Lambda para reenviar logs a Datadog.
2. Conecta esa Lambda a los grupos de logs de CloudWatch.
3. Puedes seguir la guía oficial:

➡️ [Forwarding logs from AWS CloudWatch](https://docs.datadoghq.com/logs/guide/forwarding-logs-from-aws-cloudwatch/)

---

## 🧪 Verificación

Una vez configurado el rol, en unos minutos deberías poder ver:

- Métricas en **Metrics Explorer** (prefijo `aws.`)
- Recursos en **Infrastructure > AWS**
- Dashboards en **Dashboards > AWS**

---

## 📚 Recursos útiles

- [Documentación oficial de AWS + Datadog](https://docs.datadoghq.com/integrations/amazon_web_services/)
- [Política mínima de permisos (GitHub)](https://github.com/DataDog/datadog-cloudformation-resources/blob/master/aws/datadog-integration-role.yaml)

---

## 🧹 Eliminar infraestructura

```bash
terraform destroy
```
Eliminar de cloudformation la pila, en iam deja el rol que creo por defecto.

---

### MIT License
### Copyright (c) 2025 Jose Magariño
### See LICENSE file for more details.
