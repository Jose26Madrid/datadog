# Monitoreo de AWS con Datadog (sin agentes)

Este archivo describe cómo integrar Datadog con tu cuenta de AWS para monitoreo completo **sin instalar agentes**, usando únicamente la integración oficial basada en CloudWatch.

---

## 🚀 Paso 1: Crear cuenta en Datadog

1. Visita [https://www.datadoghq.com](https://www.datadoghq.com)
2. Haz clic en **“Free Trial”** (prueba gratuita).
3. Regístrate con tu email (no se necesita tarjeta de crédito).
4. Elige **"AWS"** como fuente de datos cuando se te pregunte.

---

## 🔗 Paso 2: Conectar tu cuenta de AWS a Datadog

1. En Datadog, ve a **Integrations > Integrations**.
2. Busca **"Amazon Web Services"** y haz clic.
3. Pulsa **“Install Integration”**.
4. Utiliza el botón para **crear el rol IAM automáticamente**:
   - Te abrirá un link de CloudFormation.
   - Haz clic en **"Create Stack"** y luego **"Submit"**.
   - Se creará un **rol IAM de solo lectura** confiado por Datadog.

🛡️ Seguridad: Datadog sólo podrá leer métricas y metadatos. No puede modificar recursos.

---

## ⏱ Paso 3: Esperar unos minutos

- Datadog comenzará a recibir métricas desde **CloudWatch**.
- Verás tus servicios en:
  - `Infrastructure > Host Map`
  - `Dashboards > AWS Overview`
  - `Monitors` para alertas como uso de CPU, errores, etc.

---

## 📊 Paso 4: Explorar dashboards preconfigurados

Datadog genera automáticamente dashboards para servicios comunes:

- EC2
- RDS
- Lambda
- ELB
- S3

Puedes personalizarlos o crear los tuyos propios.

---

## 🔔 Paso 5: Crear alertas básicas

1. Ir a **Monitors > New Monitor**
2. Seleccionar una métrica (ej: `AWS/EC2 CPUUtilization`)
3. Definir umbral (ej: CPU > 85% durante 5 minutos)
4. Especificar email o canal de Slack para alertas

---

## ✅ ¡Listo!

Ya estás monitoreando tu infraestructura de AWS usando Datadog sin necesidad de instalar agentes.

### MIT License
### Copyright (c) 2025 Jose Magariño
### See LICENSE file for more details.
