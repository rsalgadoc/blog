---
title: "Día 1: Automatizando mi Organización de AWS con CloudFormation"
date: 2026-04-08
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
Este template crea una organización básica de forma económica y profesional con la siguiente estructura.

```shell
Root
├── Security
├── Infrastructure
└── Workloads
    ├── Sandbox
    ├── Dev
    └── Prod
```

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Estructura completa de AWS Organization para equipo DevOps pequeño - Con OUs y SCPs recomendados'

Resources:

  # 1. La Organization (se crea solo una vez)
  Organization:
    Type: AWS::Organizations::Organization
    Properties:
      FeatureSet: ALL

  # 2. Organizational Units (OUs)
  SecurityOU:
    Type: AWS::Organizations::OrganizationalUnit
    Properties:
      Name: Security
      ParentId: !GetAtt Organization.RootId

  InfrastructureOU:
    Type: AWS::Organizations::OrganizationalUnit
    Properties:
      Name: Infrastructure
      ParentId: !GetAtt Organization.RootId

  WorkloadsOU:
    Type: AWS::Organizations::OrganizationalUnit
    Properties:
      Name: Workloads
      ParentId: !GetAtt Organization.RootId

  SandboxOU:
    Type: AWS::Organizations::OrganizationalUnit
    Properties:
      Name: Sandbox
      ParentId: !Ref WorkloadsOU

  DevOU:
    Type: AWS::Organizations::OrganizationalUnit
    Properties:
      Name: Dev
      ParentId: !Ref WorkloadsOU

  ProdOU:
    Type: AWS::Organizations::OrganizationalUnit
    Properties:
      Name: Prod
      ParentId: !Ref WorkloadsOU

  # 3. Cuentas miembro (emails hardcodeados - cámbialos antes de desplegar)
  SecurityAuditAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: security-audit
      Email: security-audit@tu-dominio.com
      ParentIds: 
        - !Ref SecurityOU
      RoleName: OrganizationAccountAccessRole

  # Cuenta para centralizar Redes (Transit Gateway, Shared VPC, VPN)
  NetworkHubAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: network-hub
      Email: network-hub@tu-dominio.com # Cambia esto
      ParentIds: 
        - !Ref InfrastructureOU
      RoleName: OrganizationAccountAccessRole

  SharedServicesAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: shared-services
      Email: shared-services@tu-dominio.com
      ParentIds: 
        - !Ref InfrastructureOU
      RoleName: OrganizationAccountAccessRole

  SandboxAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: sandbox
      Email: sandbox@tu-dominio.com
      ParentIds: 
        - !Ref SandboxOU
      RoleName: OrganizationAccountAccessRole

  DevAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: dev
      Email: dev@tu-dominio.com
      ParentIds: 
        - !Ref DevOU
      RoleName: OrganizationAccountAccessRole

  ProdAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: prod
      Email: prod@tu-dominio.com
      ParentIds: 
        - !Ref ProdOU
      RoleName: OrganizationAccountAccessRole

```

### 🚀 Cómo desplegarlo
Guarda el código anterior como organization-structure.yaml.
Ejecuta el siguiente comando desde tu terminal (con AWS CLI configurado):

```shell
aws cloudformation create-stack \
  --stack-name AWS-Organization-Structure \
  --template-body file://organization-structure.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

O usa la consola de CloudFormation.

### 📂 Código Adjunto
Puedes encontrar el template completo y scripts adicionales en mi repositorio:
[Ver archivos en GitHub]({{ site.baseurl }}/code-samples/2026-04-08/organization-structure.yaml)