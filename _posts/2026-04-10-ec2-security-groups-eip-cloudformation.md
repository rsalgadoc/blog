---
title: "Día 2: Asegurando tu EC2 con Security Groups e Elastic IP"
date: 2026-04-10
tags: 
  - AWS
  - CloudFormation
  - EC2
  - Security
toc: true
---

# 🔒 Día 2: Asegurar tu Instancia EC2

Ayer creamos nuestra primera instancia EC2, pero sin protección. Hoy aprenderemos a asegurar tu servidor con **Security Groups**, **Elastic IP** y **parámetros reutilizables** en CloudFormation. El objetivo es tener infraestructura profesional y flexible.

### ¿Por qué es importante esto?
Exponer una instancia EC2 sin controles de tráfico es un riesgo de seguridad. Usar Security Groups y Elastic IP nos permite:
1. Controlar exactamente qué tráfico entra y sale de nuestros servidores.
2. Tener una IP pública estable (Elastic IP) incluso si reiniciamos la instancia.
3. Hacer templates reutilizables con parámetros personalizables.
4. Documentar la infraestructura con Tags y Outputs.

---

### 🛠️ El Código (CloudFormation)
Este template es más complejo y profesional que el del Día 1.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: EC2 Instance con Elastic IP y Security Groups en VPC

Parameters:
  Environment:
    Description: Entorno (dev, test, prod)
    Type: String
    Default: dev
    AllowedValues: [dev, test, prod]

  SecurityGroupDescription:
    Description: Descripción del Security Group del servidor
    Type: String
    Default: "Security group for the web/application server"

  SSHAllowedCIDR:
    Description: CIDR desde donde se permite SSH (recomendado restringirlo)
    Type: String
    Default: "0.0.0.0/0"   # Cambia esto a tu IP pública /32 en producción

  VpcId:
    Description: ID de la VPC donde se desplegará la instancia
    Type: AWS::EC2::VPC::Id

  SubnetId:
    Description: ID de la Subnet pública donde se lanzará la instancia
    Type: AWS::EC2::Subnet::Id

  ImageId:
    Description: AMI ID (Linux)
    Type: String
    Default: "ami-0ea87431b78a82070"

Resources:
  # Security Group para SSH
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref SSHAllowedCIDR
      Tags:
        - Key: Name
          Value: !Sub "${Environment}-SSH-SG"

  # Security Group para el servidor (HTTP + SSH restringido)
  ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Ref SecurityGroupDescription
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub "${Environment}-Server-SG"

  # Instancia EC2
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref ImageId
      InstanceType: t3.micro
      SubnetId: !Ref SubnetId
      SecurityGroupIds:
        - !Ref SSHSecurityGroup
        - !Ref ServerSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub "${Environment}-MyInstance"
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeSize: 8
            VolumeType: gp3
            DeleteOnTermination: true
    DependsOn: MyEIP

  # Elastic IP (en VPC)
  MyEIP:
    Type: AWS::EC2::EIP
    Properties:
      Domain: vpc
      Tags:
        - Key: Name
          Value: !Sub "${Environment}-MyEIP"

  # Asociación del Elastic IP a la instancia
  MyEIPAssociation:
    Type: AWS::EC2::EIPAssociation
    Properties:
      AllocationId: !GetAtt MyEIP.AllocationId
      InstanceId: !Ref MyInstance

Outputs:
  InstanceId:
    Description: ID de la instancia EC2
    Value: !Ref MyInstance

  PublicIPAddress:
    Description: Elastic IP asignada a la instancia
    Value: !Ref MyEIP

  SSHSecurityGroupId:
    Description: Security Group ID para SSH
    Value: !Ref SSHSecurityGroup

  ServerSecurityGroupId:
    Description: Security Group ID del servidor
    Value: !Ref ServerSecurityGroup
```

---

### 📚 Conceptos Nuevos Explicados

#### 1. **Parameters** - Plantillas Reutilizables
Los `Parameters` permiten que CloudFormation te pida valores al desplegar, sin necesidad de editar el YAML:

```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, test, prod]
```

Esto significa: "Pregunta al usuario por el entorno, con valores permitidos."

#### 2. **Security Groups** - Control de Tráfico
Los Security Groups actúan como firewalls. Aquí creamos dos grupos:

- **SSHSecurityGroup**: Permite SSH (puerto 22) desde la IP que especifiques
- **ServerSecurityGroup**: Permite HTTP (puerto 80) desde cualquier lugar

```yaml
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: !Ref SSHAllowedCIDR
```

⚠️ **Importante**: En producción, cambia `SSHAllowedCIDR` de `0.0.0.0/0` a tu IP pública + `/32` para mayor seguridad.

#### 3. **Elastic IP** - IP Estática en la Nube
Las IPs públicas normales cambian si reinicia la instancia. Elastic IP es permanente:

```yaml
MyEIP:
  Type: AWS::EC2::EIP
  Properties:
    Domain: vpc
```

Luego la asociamos a la instancia con `EIPAssociation`.

#### 4. **BlockDeviceMappings** - Configuración del Volumen
Define el tamaño y tipo de almacenamiento:

```yaml
BlockDeviceMappings:
  - DeviceName: /dev/xvda
    Ebs:
      VolumeSize: 8        # 8 GB
      VolumeType: gp3      # General Purpose 3 (más nuevo)
      DeleteOnTermination: true
```

#### 5. **Tags** - Organización
Los Tags ayudan a identificar recursos:

```yaml
Tags:
  - Key: Name
    Value: !Sub "${Environment}-MyInstance"
```

#### 6. **Outputs** - Resultados del Despliegue
Al final del template, especificamos qué información queremos ver después de desplegar:

```yaml
Outputs:
  PublicIPAddress:
    Description: Elastic IP asignada a la instancia
    Value: !Ref MyEIP
```

---

### 🚀 Cómo Desplegarlo

Guarda el código como `ec2-with-sg-eip.yaml`.

```shell
aws cloudformation create-stack \
  --stack-name Mi-EC2-Segura \
  --template-body file://ec2-with-sg-eip.yaml
```

CloudFormation te pedirá interactivamente:
- VPC ID
- Subnet ID
- Environment (dev/test/prod)
- SSH CIDR (opcional, por defecto 0.0.0.0/0)

O especifica los parámetros directamente:

```shell
aws cloudformation create-stack \
  --stack-name Mi-EC2-Segura \
  --template-body file://ec2-with-sg-eip.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=dev \
    ParameterKey=VpcId,ParameterValue=vpc-xxxxx \
    ParameterKey=SubnetId,ParameterValue=subnet-xxxxx
```

---

### 📂 Código Adjunto
Puedes encontrar el template completo en mi repositorio:
[Ver archivos en GitHub]({{ site.baseurl }}/code-samples/2026-04-10/ec2-with-sg-eip.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="NuOohtLHyzU" provider="youtube" %}

---

### 💡 Próximos pasos

- Crear una aplicación en la instancia y desplegarla automáticamente con UserData
