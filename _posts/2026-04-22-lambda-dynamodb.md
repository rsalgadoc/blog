---
title: "Día 14: AWS Lambda + DynamoDB: Crea una API simple para guardar y leer datos"
date: 2026-04-22
tags:
  - CloudFormation
  - Lambda
  - DynamoDB
  - Python
  - Serverless
toc: true
---

# ⚡ Día 14: AWS Lambda + DynamoDB — ¡Tu primera base de datos serverless!

Hoy daremos el salto definitivo hacia las aplicaciones reales. No solo vamos a procesar datos, sino que vamos a **persistirlos**. Vamos a conectar nuestra Lambda con **Amazon DynamoDB**, la base de datos NoSQL clave-valor de AWS que escala a niveles increíbles con latencia de milisegundos.

Esta combinación es el estándar de oro para aplicaciones modernas donde no quieres gestionar servidores ni preocuparte por el aprovisionamiento de espacio.

### ¿Por qué usar DynamoDB con Lambda?

* **Totalmente Serverless**: No hay instancias que encender ni parches que aplicar.
* **Escalabilidad masiva**: Maneja desde un par de peticiones hasta millones por segundo sin despeinarse.
* **Pago por uso**: Ideal para proyectos que están empezando o tienen tráfico irregular.
* **Integración nativa**: Gracias a la librería `boto3` en Python, interactuar con las tablas es sumamente sencillo.

### 🛠️ El Código (CloudFormation)

En este despliegue vamos a crear:
- Una **Tabla de DynamoDB** simple (con una clave de partición).
- Una **Función Lambda** que puede escribir y leer de esa tabla.
- Un **Rol IAM** con permisos específicos para que la Lambda "hable" con DynamoDB.
- Un **API Gateway** para que podamos probarlo desde el navegador.

#### 1. Tabla de DynamoDB y Rol de Ejecución

```yaml
  MiTablaDatos:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: TablaUsuariosDia14
      AttributeDefinitions:
        - AttributeName: usuarioId
          AttributeType: S
      KeySchema:
        - AttributeName: usuarioId
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ://amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: DynamoDBAccess
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - dynamodb:PutItem
                  - dynamodb:GetItem
                Resource: !GetAtt MiTablaDatos.Arn
```

#### 2. La función Lambda (Lógica de Guardar y Leer)

```yaml
  LambdaCRUD:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: Lambda-Dynamo-Simple
      Runtime: python3.12
      Role: !GetAtt LambdaExecutionRole.Arn
      Handler: index.lambda_handler
      Code:
        ZipFile: |
          import json
          import boto3
          import os

          dynamo = boto3.resource('dynamodb')
          table = dynamo.Table('TablaUsuariosDia14')

          def lambda_handler(event, context):
              method = event['requestContext']['http']['method']
              
              if method == 'POST':
                  # Guardar un dato (enviado en el body)
                  body = json.loads(event.get('body', '{}'))
                  table.put_item(Item={'usuarioId': body['id'], 'nombre': body['nombre']})
                  return {'statusCode': 201, 'body': json.dumps({'mensaje': 'Usuario guardado!'})}
              
              elif method == 'GET':
                  # Leer un dato (vía query string ?id=123)
                  user_id = event.get('queryStringParameters', {}).get('id')
                  if not user_id: return {'statusCode': 400, 'body': 'Falta el id'}
                  
                  response = table.get_item(Key={'usuarioId': user_id})
                  return {'statusCode': 200, 'body': json.dumps(response.get('Item', 'No encontrado'))}
```

#### 3. API Gateway HTTP + Permisos

```yaml
  MiAPI:
    Type: AWS::ApiGatewayV2::Api
    Properties:
      Name: APIDynamoDB
      ProtocolType: HTTP
      Target: !GetAtt LambdaCRUD.Arn

  AllowAPIGatewayToInvoke:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref LambdaCRUD
      Action: lambda:InvokeFunction
      Principal: ://amazonaws.com
      SourceArn: !Sub "arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${MiAPI}/*"
```

### 📚 Conceptos Nuevos Explicados

#### 1. Partition Key (usuarioId)
Es el identificador único en DynamoDB. En este ejemplo, definimos `usuarioId` como un String (`S`). Es lo que DynamoDB usa para distribuir los datos internamente.

#### 2. BillingMode: PAY_PER_REQUEST
Configuramos la tabla para que no tenga un costo fijo. Solo pagas por cada lectura o escritura real que realices. ¡Perfecto para pruebas!

#### 3. Boto3 (SDK de AWS para Python)
Es la librería que importamos dentro de la Lambda. `boto3.resource('dynamodb')` nos permite interactuar con la base de datos usando métodos simples como `.put_item()` y `.get_item()`.

#### 4. IAM Policies Específicas
A diferencia del post anterior, aquí agregamos una política manual para que la Lambda tenga permiso de acceder **solo** a nuestra tabla específica, siguiendo el principio de "mínimo privilegio".

### 🚀 Cómo Desplegarlo

Usa el comando habitual para crear el stack:

```shell
aws cloudformation deploy \
  --stack-name Mi-Primera-BD-Serverless \
  --template-file 22-lambda-dynamodb.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 📈 Cómo probar tu API con Base de Datos

1. Obtén la **Invoke URL** desde la consola de API Gateway.
2. **Para Guardar (POST):** Usa Postman o cURL:
   ```bash
   curl -X POST https://o008pgtcbd.execute-api.us-east-1.amazonaws.com/prod/ \
   -H "Content-Type: application/json" \
   -d '{"id": "1", "nombre": "Andrés"}'
   ```
3. **Para Leer (GET):** Simplemente abre en tu navegador:
   `https://o008pgtcbd.execute-api.us-east-1.amazonaws.com/prod/?id=1`

**Ejemplo de respuesta esperada (GET):**
```JSON
{
  "usuarioId": "1",
  "nombre": "Andrés"
}
```

### 📂 Código Adjunto
Puedes encontrar el template y el código de la función aquí:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/22-lambda-dynamodb.yaml)

---

### 🎥 Video Tutorial
Mira cómo configurar la tabla y probar la API en vivo:

{% include video.html id="YxK1D_rtf6Q" provider="youtube" %}

---

### 💡 Próximos pasos
- Optimizando costos y rendimiento en AWS Lambda: Memory, Timeout y Concurrency (Mejores prácticas de configuración y monitoreo)
