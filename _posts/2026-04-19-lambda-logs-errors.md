---
title: "Día 11: Manejo de errores y logs en AWS Lambda con CloudWatch Logs"
date: 2026-04-19
tags:
  - CloudFormation
  - Lambda
  - CloudWatch
  - Python
toc: true
---

# 🚀 Día 11: Debugging en Lambda — Logs y manejo de errores con CloudWatch

Ahora que ya sabes crear funciones Lambda y exponerlas con API Gateway, es fundamental aprender a **ver qué está pasando dentro** de tu código. Hoy vamos a enfocarnos en el **manejo de errores** y el uso correcto de **CloudWatch Logs** para hacer debugging profesional.

### ¿Por qué es importante manejar logs y errores en Lambda?

* Saber exactamente qué ocurrió cuando algo falla.
* Detectar y solucionar problemas rápidamente.
* Evitar que las funciones fallen silenciosamente.
* Controlar costos (los logs ocupan almacenamiento).
* Aplicar buenas prácticas de observabilidad en serverless.

### 🛠️ El Código (CloudFormation)

En este ejemplo crearemos una función Lambda con buen logging, manejo de excepciones y configuración de retención de logs.

#### 1. Rol de ejecución IAM (con permisos completos para logs)

```yaml
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
      - PolicyName: CloudWatchLogsPolicy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - logs:CreateLogGroup
                - logs:CreateLogStream
                - logs:PutLogEvents
              Resource: "*"
```

#### 2. La función Lambda - Versión Python (con logging y manejo de errores)

```yaml
MiLambdaConLogs:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: LambdaConLogs-Python
    Runtime: python3.12
    Role: !GetAtt LambdaExecutionRole.Arn
    Handler: index.lambda_handler
    Timeout: 30
    Code:
      ZipFile: |
        import json
        import logging

        # Configuración básica de logging
        logger = logging.getLogger()
        logger.setLevel(logging.INFO)

        def lambda_handler(event, context):
            logger.info("=== Inicio de la función Lambda ===")
            logger.info(f"Request ID: {context.aws_request_id}")
            logger.info(f"Evento recibido: {json.dumps(event, default=str)}")
            
            try:
                # Simulamos posible error para demostración
                if "error" in event.get("queryStringParameters", {}):
                    raise ValueError("Error simulado solicitado por el usuario")
                
                response_body = {
                    "mensaje": "¡La función se ejecutó correctamente!",
                    "version": "Día 11",
                    "request_id": context.aws_request_id,
                    "timestamp": context.get_remaining_time_in_millis()
                }
                
                logger.info("Función ejecutada exitosamente")
                return {
                    "statusCode": 200,
                    "headers": {
                        "Content-Type": "application/json",
                        "Access-Control-Allow-Origin": "*"
                    },
                    "body": json.dumps(response_body)
                }
                
            except Exception as e:
                logger.error(f"Error en la función: {str(e)}", exc_info=True)
                
                return {
                    "statusCode": 500,
                    "headers": {
                        "Content-Type": "application/json",
                        "Access-Control-Allow-Origin": "*"
                    },
                    "body": json.dumps({
                        "mensaje": "Ocurrió un error interno",
                        "error": str(e)
                    })
                }
```

#### 3. Configuración de retención de logs (¡Nuevo!)

```yaml
  MiLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Sub "/aws/lambda/${MiLambdaConLogs}"
      RetentionInDays: 30        # Opciones comunes: 7, 14, 30, 90, 365
```

### 📚 Conceptos Nuevos Explicados
#### 1. CloudWatch Logs
Servicio donde Lambda guarda automáticamente todos los print(), console.log(), logger.info(), etc.

#### 2. Niveles de logging
* `INFO` → Información normal (recomendado para producción)
* `ERROR` → Errores que deben ser investigados
* `DEBUG` → Solo para desarrollo (puedes activarlo temporalmente)

#### 3. Manejo de excepciones (try/except)
Siempre debes capturar errores y devolver una respuesta HTTP adecuada (500 en caso de fallo).
#### 4. Retención de logs (RetentionInDays)
Por defecto, los logs se guardan **indefinidamente**, lo que puede generar costos altos.
Recomendaciones comunes:

* **7 días** → Entornos de desarrollo
* **30 días** → Entornos de producción
* **90 o 365 días** → Cumplimiento normativo

### 🚀 Cómo Desplegarlo
Actualiza tu stack de CloudFormation (o crea uno nuevo si prefieres):

```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://19-lambda-logs-errors.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 📈 Cómo probar y ver los logs

1. Ve a la consola de **AWS Lambda** → busca tu función (`LambdaConLogs-Python`).
2. Haz clic en la pestaña **Test**.
3. Haz clic en Create new test event (o Create test event).
4. Ponle un nombre al evento, por ejemplo: TestError
5. Pega uno de estos dos eventos de prueba:

**Evento para prueba exitosa:**
```json
{
  "mensaje": "prueba normal"
}
```
Evento para prueba con error:
```json
{
  "queryStringParameters": {
    "error": "true"
  }
}
```
6. Haz clic en Save y luego en **Test**.


* En la prueba normal deberías ver statusCode 200.
* En la prueba con error deberías ver statusCode 500.


7. Ve a **Monitor → View logs in CloudWatch** para ver los logs detallados.

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
