---
title: "Día 1: Desplegando una Instancia EC2 con CloudFormation"
date: 2026-04-09
tags: 
  - AWS
  - CloudFormation
  - EC2
toc: true
---

# ☁️ Día 1: Tu Primera Instancia EC2

Es momento de crear tu primer servidor en la nube. Hoy aprenderemos a desplegar una **instancia EC2** de forma programática usando CloudFormation de manera simple y efectiva.

### ¿Por qué usar CloudFormation para esto?
Crear instancias manualmente en la consola de AWS es tedioso y propenso a errores. Usar **Infraestructura como Código (IaC)** nos permite:
1. Reproducir la misma configuración en segundos.
2. Versionear y documentar nuestros cambios.
3. Automatizar deployments en múltiples entornos.

---

### 🛠️ El Código (CloudFormation)
Este template crea una instancia EC2 básica con configuración mínima pero funcional.

```yaml
---
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0ea87431b78a82070
      InstanceType: t3.micro
```

**Desglose del template:**
- **Type**: `AWS::EC2::Instance` → Especifica que estamos creando una instancia EC2
- **AvailabilityZone**: `us-east-1a` → La zona de disponibilidad donde se lanzará la instancia
- **ImageId**: `ami-0ea87431b78a82070` → AMI de Amazon Linux 2 (cambia según tu región)
- **InstanceType**: `t3.micro` → Tipo de instancia (elegible para Free Tier)

---

### 🚀 Cómo desplegarlo
Guarda el código anterior como `just-ec2.yaml`.
Ejecuta el siguiente comando desde tu terminal (con AWS CLI configurado):

```shell
aws cloudformation create-stack \
  --stack-name Mi-Primera-EC2 \
  --template-body file://just-ec2.yaml
```

O a través de la consola de CloudFormation en AWS.

### 📂 Código Adjunto
Puedes encontrar el template completo y más ejemplos en mi repositorio:
[Ver archivos en GitHub]({{ site.baseurl }}/code-samples/2026-04-09/just-ec2.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="H_yoRyEamiU" provider="youtube" %}

---

### 💡 Próximos pasos
- Agregar un grupo de seguridad para controlar el tráfico
- Configurar variables de entorno
- Escalar a múltiples instancias
