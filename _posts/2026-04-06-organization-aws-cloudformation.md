---
title: "Día 1: Automatizando mi Organización de AWS con CloudFormation"
date: 2026-04-06
tags: 
  - AWS
  - CloudFormation
toc: true
---

# ☁️ Día 1: Estructura Base de AWS
Para escalar en la nube, lo primero es el orden. Hoy aprenderemos a desplegar una **AWS Organization** de forma programática.

### ¿Por qué usar CloudFormation para esto?
Configurar cuentas manualmente en la consola de AWS es propenso a errores. Usar **Infraestructura como Código (IaC)** nos permite:
1. Tener un registro histórico de cambios.
2. Desplegar Unidades Organizativas (OUs) en segundos.
3. Mantener la consistencia en múltiples entornos.

---

### 🛠️ El Código (CloudFormation)
Este template crea una organización básica y una Unidad Organizativa llamada `Seguridad`.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Template para crear una AWS Organization básica'

Resources:
  MyOrganization:
    Type: 'AWS::Organizations::Organization'
    Properties:
      FeatureSet: ALL

  SecurityOU:
    Type: 'AWS::Organizations::OrganizationalUnit'
    Properties:
      Name: 'Seguridad'
      ParentId: !GetAtt MyOrganization.RootId
```

### 🚀 Cómo desplegarlo
Guarda el código anterior como org-base.yaml.
Ejecuta el siguiente comando desde tu terminal (con AWS CLI configurado):

```shell
aws cloudformation create-stack \
  --stack-name MiOrganizacion \
  --template-body file://org-base.yaml
```

### 📂 Código Adjunto
Puedes encontrar el template completo y scripts adicionales en mi repositorio:
[Ver archivos en GitHub]({{ site.url }}/code-samples/post-org-aws/org-base.yaml)