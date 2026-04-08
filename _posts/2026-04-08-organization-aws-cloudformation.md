---
title: "Día 1: Automatizando mi Organización de AWS con CloudFormation"
date: 2026-04-07
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
      FeatureSet: ALL_FEATURES

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

  SandboxAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: sandbox
      Email: sandbox@tu-dominio.com
      ParentIds: 
        - !Ref SandboxOU

  DevAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: dev
      Email: dev@tu-dominio.com
      ParentIds: 
        - !Ref DevOU

  ProdAccount:
    Type: AWS::Organizations::Account
    Properties:
      AccountName: prod
      Email: prod@tu-dominio.com
      ParentIds: 
        - !Ref ProdOU

  # 4. SCPs recomendados (adjuntados al Root o a OUs específicas)

  # SCP 1: Proteger la cuenta Management (Root) - Impide acciones con usuario root
  ProtectManagementAccountSCP:
    Type: AWS::Organizations::Policy
    Properties:
      Name: Protect-Management-Account
      Description: Protege la cuenta raíz de acciones peligrosas usando credenciales root
      Type: SERVICE_CONTROL_POLICY
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: ProtectRootUser
            Effect: Deny
            Action: "*"
            Resource: "*"
            Condition:
              StringEquals:
                aws:PrincipalARN: "arn:aws:iam::${aws:accountId}:root"
      TargetIds:
        - !GetAtt Organization.RootId

  # SCP 2: Restringir regiones (solo permite las que tú uses)
  RestrictRegionsSCP:
    Type: AWS::Organizations::Policy
    Properties:
      Name: Restrict-Regions
      Description: Solo permite usar regiones aprobadas (us-east-1 y sa-east-1)
      Type: SERVICE_CONTROL_POLICY
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: DenyUnsupportedRegions
            Effect: Deny
            Action: "*"
            Resource: "*"
            Condition:
              StringNotEquals:
                aws:RequestedRegion:
                  - us-east-1
                  - sa-east-1
      TargetIds:
        - !GetAtt Organization.RootId

  # SCP 3: Forzar tagging obligatorio (CostCenter y Environment)
  RequireTagsSCP:
    Type: AWS::Organizations::Policy
    Properties:
      Name: Require-Tags
      Description: Obliga a usar tags CostCenter y Environment en recursos clave
      Type: SERVICE_CONTROL_POLICY
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: RequireCostCenterAndEnvironment
            Effect: Deny
            Action:
              - ec2:RunInstances
              - s3:CreateBucket
              - rds:CreateDBInstance
              - lambda:CreateFunction
              - dynamodb:CreateTable
            Resource: "*"
            Condition:
              Null:
                aws:RequestTag/CostCenter: "true"
                aws:RequestTag/Environment: "true"
      TargetIds:
        - !Ref WorkloadsOU   # Solo se aplica a Workloads (puedes cambiarlo)

  # SCP 4: Evitar que las cuentas salgan de la Organization
  PreventLeavingOrgSCP:
    Type: AWS::Organizations::Policy
    Properties:
      Name: Prevent-Leaving-Organization
      Description: Impide que las cuentas miembro abandonen la Organization
      Type: SERVICE_CONTROL_POLICY
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: DenyLeaveOrganization
            Effect: Deny
            Action: organizations:LeaveOrganization
            Resource: "*"
      TargetIds:
        - !GetAtt Organization.RootId

  # SCP 5: Denegar recursos caros en Sandbox y Dev (control de costos)
  DenyExpensiveResourcesSCP:
    Type: AWS::Organizations::Policy
    Properties:
      Name: Deny-Expensive-Resources
      Description: Bloquea instancias y recursos muy caros en entornos no productivos
      Type: SERVICE_CONTROL_POLICY
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: DenyHighCostInstances
            Effect: Deny
            Action:
              - ec2:RunInstances
              - rds:CreateDBInstance
            Resource: "*"
            Condition:
              StringEquals:
                ec2:InstanceType:
                  - p3.2xlarge
                  - g4dn.12xlarge
                  - m5.24xlarge
                  - r5.24xlarge
      TargetIds:
        - !Ref SandboxOU
        - !Ref DevOU
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