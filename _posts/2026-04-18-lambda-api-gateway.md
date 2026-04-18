---
title: "Día 10: AWS Lambda + Amazon API Gateway: Crea tu primera API REST en minutos"
date: 2026-04-18
tags:
  - CloudFormation
  - Lambda
  - API Gateway
  - Python
toc: true
---

# 🚀 Día 10: Exponiendo tu Lambda al mundo — ¡Tu primera API REST serverless!

Hoy vamos a dar un paso muy importante: vamos a exponer nuestra función Lambda a través de **Amazon API Gateway** para que pueda ser consumida como una **API REST** real desde cualquier lugar (frontend, móvil, Postman, etc.).

Esta combinación (Lambda + API Gateway) es una de las más usadas para crear backends serverless completos.

### ¿Por qué usar API Gateway con Lambda?

* **Convierte tu función en una API**: Accesible vía HTTP (GET, POST, etc.).
* **Manejo automático de solicitudes**: Se encarga de routing, validación básica y respuestas HTTP.
* **Seguridad integrada**: Soporta claves API, Cognito, IAM, etc. (lo veremos más adelante).
* **Escalabilidad y bajo costo**: Solo pagas por las solicitudes que recibe la API.
* Ideal para crear microservicios, CRUDs simples o backends para aplicaciones web/móviles.

### 🛠️ El Código (CloudFormation)

Vamos a crear:
- La misma función Lambda del “Hola Mundo” (la reutilizamos)
- Un API Gateway HTTP (el más simple y moderno)
- Una integración entre API Gateway y Lambda
- Un deployment y un stage para que la API esté pública

#### 1. Rol de ejecución IAM (reutilizamos el del Día 8)

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
```

#### 2. La función Lambda (reutilizamos la del Día 8 - Python)

```yaml
MiPrimeraLambdaPython:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: MiPrimeraAPILambda-Python
    Runtime: python3.12
    Role: !GetAtt LambdaExecutionRole.Arn
    Handler: index.lambda_handler
    Timeout: 30
    Code:
      ZipFile: |
        import json

        def lambda_handler(event, context):
            print("Evento recibido desde API Gateway:", json.dumps(event, indent=2))
            
            response_body = {
                'mensaje': '¡Hola desde mi primera API con Lambda y API Gateway! 🚀',
                'version': 'Día 10',
                'timestamp': context.get('aws_request_id'),
                'method': event.get('requestContext', {}).get('http', {}).get('method', 'UNKNOWN'),
                'path': event.get('requestContext', {}).get('http', {}).get('path', '/')
            }
            
            return {
                'statusCode': 200,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*'   # Para permitir CORS (útil en desarrollo)
                },
                'body': json.dumps(response_body)
            }
```
#### 3. API Gateway HTTP + Integración con Lambda

```yaml
MiAPI:
  Type: AWS::ApiGatewayV2::Api
  Properties:
    Name: MiPrimeraAPI
    ProtocolType: HTTP
    Target: !Sub "arn:aws:lambda:$${AWS::Region}:$${AWS::AccountId}:function:${MiPrimeraLambdaPython}"

MiStage:
  Type: AWS::ApiGatewayV2::Stage
  Properties:
    ApiId: !Ref MiAPI
    Name: prod
    AutoDeploy: true
```
#### 4. Permiso para que API Gateway invoque la Lambda

```yaml
AllowAPIGatewayToInvokeLambda:
  Type: AWS::Lambda::Permission
  Properties:
    FunctionName: !Ref MiPrimeraLambdaPython
    Action: lambda:InvokeFunction
    Principal: apigateway.amazonaws.com
    SourceArn: !Sub "arn:aws:execute-api:$${AWS::Region}:$${AWS::AccountId}:${MiAPI}/*/*/*"
```

### 📚 Conceptos Nuevos Explicados
#### 1. Amazon API Gateway
Servicio gestionado que permite crear, publicar y proteger APIs a escala. En este caso usamos la versión HTTP API (más simple, rápida y barata que REST API).
#### 2. Integración Lambda (Proxy integration)
Al poner Target en el recurso AWS::ApiGatewayV2::Api, estamos usando integración proxy: API Gateway pasa toda la solicitud directamente a Lambda y devuelve la respuesta tal cual.
#### 3. Stage (prod)
Es el entorno donde se publica la API. AutoDeploy: true hace que cualquier cambio se despliegue automáticamente.
#### 4. Lambda Permission
Necesario para que API Gateway tenga permiso de invocar tu función Lambda.

### 🚀 Cómo Desplegarlo
Actualiza tu stack de CloudFormation (o crea uno nuevo si prefieres):

```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://18-lambda-api-gateway.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 📈 Cómo probar tu primera API

1. Después del despliegue, ve a la consola de API Gateway → busca “MiPrimeraAPI”.
2. En la sección Stages → haz clic en prod.
3. Copia la Invoke URL (se ve algo como https://abc123.execute-api.us-east-1.amazonaws.com/prod).
4. Abre esa URL en el navegador o en Postman.
5. Deberías ver la respuesta JSON con el mensaje “¡Hola desde mi primera API...”.

**Ejemplo de respuesta esperada:**

```JSON
{
  "mensaje": "¡Hola desde mi primera API con Lambda y API Gateway! 🚀",
  "version": "Día 10",
  "request_id": "...",
  "path": "/"
}
```

### 📂 Código Adjunto
Puedes encontrar el template completo (rol + Lambda + API Gateway + permisos) aquí:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/18-lambda-api-gateway.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="lD6TuB4khGw" provider="youtube" %}

---

### 💡 Próximos pasos
- Manejo de errores y logs en AWS Lambda con CloudWatch Logs(Debugging básico y buenas prácticas)
