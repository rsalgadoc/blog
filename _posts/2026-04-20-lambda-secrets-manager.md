---
title: "Día 12: AWS Lambda con Variables de Entorno y Secrets Manager"
date: 2026-04-20
tags:
  - CloudFormation
  - Lambda
  - Secrets Manager
  - Python
toc: true
---

# 🚀 Día 12: Configuración segura en Lambda — Variables de Entorno y Secrets Manager

Hoy vamos a aprender una de las prácticas más importantes en serverless: **cómo manejar credenciales y configuraciones sensibles** sin hardcodearlas en el código.

Usaremos **Variables de Entorno** para valores no sensibles y **AWS Secrets Manager** para almacenar secretos de forma segura (claves API, tokens, contraseñas, etc.).

### ¿Por qué usar Variables de Entorno y Secrets Manager?

* Evitar credenciales hardcodeadas (riesgo de seguridad alto).
* Separar la configuración del código.
* Facilitar la rotación de secretos sin redeployar la función.
* Cumplir con buenas prácticas de seguridad y compliance.
* Diferenciar fácilmente entre entornos (desarrollo, staging, producción).

### ⚠️ Nota importante sobre seguridad

**Nunca** debes guardar valores sensibles directamente en el template de CloudFormation.  
En este ejemplo crearemos el secreto **manualmente** en la consola de Secrets Manager y solo referenciamos su nombre desde CloudFormation. Esta es la forma recomendada en entornos reales.

### 🛠️ El Código (CloudFormation)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Lambda con Variables de Entorno y Secrets Manager (Seguro)

Resources:
  # ==================== ROL IAM ====================
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: SecretsManagerPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - secretsmanager:GetSecretValue
                Resource: !Sub "arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:lambda/api-key-prod*"

  # ==================== FUNCIÓN LAMBDA ====================
  MyLambdaWithSecrets:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: LambdaWithSecrets-Python
      Runtime: python3.12
      Role: !GetAtt LambdaExecutionRole.Arn
      Handler: index.lambda_handler
      Timeout: 30
      Environment:
        Variables:
          ENVIRONMENT: "production"
          LOG_LEVEL: "INFO"
          SECRET_NAME: "lambda/api-key-prod"   # Nombre del secreto creado manualmente
      Code:
        ZipFile: |
          import json
          import logging
          import boto3
          import os

          logger = logging.getLogger()
          logger.setLevel(os.getenv("LOG_LEVEL", "INFO"))

          def get_secret(secret_name):
              """Obtiene el secreto de forma segura desde Secrets Manager"""
              client = boto3.client('secretsmanager')
              response = client.get_secret_value(SecretId=secret_name)
              # Si el secreto está en formato string JSON, lo parseamos
              if 'SecretString' in response:
                  return json.loads(response['SecretString'])
              # Si está en binario (menos común)
              else:
                  return json.loads(response['SecretBinary'].decode('utf-8'))

          def lambda_handler(event, context):
              logger.info("=== Inicio de la función Lambda ===")
              logger.info(f"Request ID: {context.aws_request_id}")
              logger.info(f"Entorno: {os.getenv('ENVIRONMENT')}")

              try:
                  secret_name = os.getenv("SECRET_NAME")
                  secret = get_secret(secret_name)
                  
                  api_key = secret.get("api_key")
                  
                  logger.info("Secreto obtenido correctamente desde Secrets Manager")
                  # Nunca loguees el valor real del secreto en producción

                  response_body = {
                      "mensaje": "Función ejecutada correctamente con Secrets Manager",
                      "version": "Día 12",
                      "environment": os.getenv("ENVIRONMENT"),
                      "api_key_length": len(api_key) if api_key else 0,
                      "request_id": context.aws_request_id
                  }
                  
                  logger.info("Función completada exitosamente")
                  return {
                      "statusCode": 200,
                      "headers": {
                          "Content-Type": "application/json"
                      },
                      "body": json.dumps(response_body)
                  }
                  
              except Exception as e:
                  logger.error(f"Error en la función: {str(e)}", exc_info=True)
                  
                  return {
                      "statusCode": 500,
                      "headers": {
                          "Content-Type": "application/json"
                      },
                      "body": json.dumps({
                          "mensaje": "Ocurrió un error al obtener el secreto",
                          "error": str(e)
                      })
                  }
```

### 📚 Conceptos Nuevos Explicados
#### 1. Variables de Entorno (`Environment.Variables`)
Se usan para pasar configuración no sensible a la Lambda (entorno, nivel de logs, nombres, etc.).
#### 2.  AWS Secrets Manager
Servicio gestionado para almacenar y rotar secretos de forma segura. Ideal para credenciales.
#### 3.  Permisos IAM mínimos
Solo se da el permiso `secretsmanager:GetSecretValue` sobre el secreto específico (principio de menor privilegio).
#### 4.  Creación manual del secreto
Paso previo obligatorio:
Ve a la consola de **Secrets Manager** → **Store a new secret** → elige "Other type of secret" → agrega una clave `api_key` con tu valor → nómbralo exactamente `lambda/api-key-prod`.
#### 5.  Buenas prácticas de seguridad

* Nunca imprimir el valor real del secreto en los logs.
* Usar `os.getenv()` para leer variables de entorno.
* Rotar los secretos periódicamente desde Secrets Manager.


### 📌 Nota sobre cambios en Variables de Entorno
Cuando modificas las variables de entorno de una función Lambda:

* El cambio no es instantáneo.
* AWS tarda unos segundos en propagar la nueva configuración.
* Las siguientes ejecuciones después del cambio usarán el nuevo valor.
* Recomendación: Espera 10-15 segundos después de guardar el cambio antes de volver a probar.

Ejemplo: Si cambias `LOG_LEVEL` de `"INFO"` a `"DEBUG"`, los logs más detallados aparecerán en las invocaciones posteriores.

Este es un <code style="color: orange; background: black;">valor naranja</code>

### 🚀 Cómo Desplegarlo
Primero crea el secreto manualmente en Secrets Manager con el nombre `lambda/api-key-prod`.
Luego despliega el stack:

```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://20-lambda-secrets-manager.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 📈 Cómo probar tu función

1. Ve a la consola de **AWS Lambda** → busca `LambdaWithSecrets-Python`.
2. Haz clic en **Test** → crea un test event vacío (`{}`).
3. Ejecuta el test.
4. Revisa la respuesta: deberías ver `api_key_length` con un valor mayor a 0.
5. Ve a **Monitor → View logs in CloudWatch** para verificar que el secreto se leyó correctamente.

**Importante**: En los logs solo verás la longitud de la clave, nunca el valor completo.

### 📂 Código Adjunto
Puedes encontrar el template completo aquí:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/19-lambda-logs-errors.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="FKMXd_vJD80" provider="youtube" %}

---

### 💡 Próximos pasos
- AWS Lambda con Variables de Entorno y Secrets Manager
(Configuración segura sin hardcodear credenciales)
