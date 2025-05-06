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

## 🧹 Eliminar infraestructura

```bash
terraform destroy
```
Eliminar de cloudformation la pila, en iam deja el rol que creo por defecto.

---

### MIT License
### Copyright (c) 2025 Jose Magariño
### See LICENSE file for more details.
