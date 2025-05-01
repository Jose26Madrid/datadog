# Monitoreo con Datadog y despliegue de imágenes en AWS

Este proyecto demuestra cómo:

- Crear una instancia EC2 con Terraform
- Instalar el agente de Datadog
- Subir imágenes Docker a ECR
- Ejecutar contenedores y visualizar métricas en Datadog

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

## ▶️ Ejecutar contenedor para que Datadog lo detecte

```bash
sudo docker run -d --name contenedor-prueba <account>.dkr.ecr.eu-west-1.amazonaws.com/mi-imagen-ecr:latest sleep 300
```

---

## 📊 Ver métricas en Datadog

1. Ir a [Infrastructure > Containers](https://app.datadoghq.eu/infrastructure)
2. Usar filtros como:
   - `container_name:contenedor-prueba`
   - `image.name:<ecr-uri>`

---

## 🧹 Eliminar infraestructura

```bash
terraform destroy
```

---

## 📝 Notas

- Solo se ven imágenes en Datadog si están **en ejecución** como contenedor.
- Para ver imágenes almacenadas en ECR sin ejecutarlas, es necesario conectar AWS a Datadog mediante **CloudFormation**.
- La vista de vulnerabilidades requiere activar **Cloud Workload Security** (CWS).

---


### MIT License
### Copyright (c) 2025 Jose Magariño
### See LICENSE file for more details.
