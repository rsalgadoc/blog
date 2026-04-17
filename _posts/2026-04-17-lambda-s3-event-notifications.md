---
title: "Día 9: Conectando AWS Lambda con Amazon S3: Procesa archivos subidos automáticamente"
date: 2026-04-17
tags:
  - CloudFormation
  - Lambda
  - S3
  - Python
toc: true
---

# 🚀 Día 9: ¡Tu primera automatización real! Lambda + S3

Hoy damos el siguiente paso lógico en serverless: **hacer que tu función Lambda se ejecute automáticamente** cada vez que alguien sube un archivo a un bucket de **Amazon S3**. Esto es una de las combinaciones más poderosas y usadas en AWS.

### ¿Por qué conectar Lambda con S3?

* **Automatización total**: Procesa imágenes, valida documentos, genera thumbnails, extrae texto, etc. sin intervención manual.
* **Event-driven**: Lambda solo se ejecuta cuando ocurre algo (subida de archivo).
* **Escalable y barato**: Solo pagas cuando se sube un archivo.
* Muy común en pipelines de datos, procesamiento de medios y backends serverless.

### 🛠️ El Código (CloudFormation)

Vamos a crear:
- Un bucket de S3
- Una función Lambda que se dispara automáticamente al subir archivos (solo `.jpg`, `.png` y `.pdf` para evitar bucles)
- Permisos necesarios

#### 1. Rol de ejecución IAM (con permisos para leer S3)

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
      - PolicyName: S3ReadPolicy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - s3:GetObject
              Resource: !Sub "arn:aws:s3:::${MiBucket}/*"
```

#### 2. El Bucket S3

```yaml
MiBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "mi-bucket-procesamiento-${AWS::AccountId}"
    NotificationConfiguration:
      LambdaConfigurations:
        - Event: s3:ObjectCreated:*
          Function: !GetAtt MiLambdaProcesarS3.Arn
```
#### 3. Permiso para que S3 invoque la Lambda

```yaml
AllowS3ToInvokeLambda:
  Type: AWS::Lambda::Permission
  Properties:
    FunctionName: !Ref MiLambdaProcesarS3
    Action: lambda:InvokeFunction
    Principal: s3.amazonaws.com
    SourceArn: !GetAtt MiBucket.Arn
```
#### 4. La función Lambda - Versión Python

```yaml
MiLambdaProcesarS3:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: ProcesarArchivoS3-Python
    Runtime: python3.12
    Role: !GetAtt LambdaExecutionRole.Arn
    Handler: index.lambda_handler
    Timeout: 30
    Code:
      ZipFile: |
        import json
        import boto3
        import urllib.parse

        s3 = boto3.client('s3')

        def lambda_handler(event, context):
            print("Evento recibido:", json.dumps(event, indent=2))
            
            # Obtener información del archivo subido
            bucket = event['Records'][0]['s3']['bucket']['name']
            key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
            
            print(f"Archivo subido: s3://{bucket}/{key}")
            
            # Aquí puedes procesar el archivo (ejemplo: leer metadata)
            response = s3.get_object(Bucket=bucket, Key=key)
            content_type = response['ContentType']
            
            print(f"Tipo de contenido: {content_type}")
            
            return {
                'statusCode': 200,
                'body': f"Procesado correctamente el archivo {key} de tipo {content_type}"
            }
```

### 📚 Conceptos Nuevos Explicados
#### 1. Trigger de S3
Es la configuración que permite a Amazon S3 invocar automáticamente tu función Lambda cuando ocurre un evento (en este caso `s3:ObjectCreated:*`).
#### 2. NotificationConfiguration
Se define dentro del recurso `AWS::S3::Bucket` para indicar qué eventos deben notificar a Lambda.
#### 3. Lambda Permission (AWS::Lambda::Permission)
Es obligatorio. Le da permiso explícito a S3 para invocar tu función Lambda. Sin esto, el trigger no funcionará aunque esté configurado.
#### 4. Evento de S3
El `event` que recibe Lambda contiene información del bucket y la clave del objeto subido. Usamos `boto3` (Python) para interactuar con S3.

### 🚀 Cómo Desplegarlo
Actualiza tu stack de CloudFormation (o crea uno nuevo si prefieres):

```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://17-lambda-s3-event-notifications.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 📈 Cómo probar tu función con S3

1. Ve a la consola de **S3** → busca tu bucket (`mi-bucket-procesamiento-...`).
2. Sube un archivo `.jpg`, `.png` o `.pdf`.
3. Ve a la consola de **Lambda** → busca tu función (`ProcesarArchivoS3-Python`).
4. En la sección **Monitor** o **Invocaciones** verás la ejecución.
5. Revisa los **logs** en CloudWatch Logs para ver el mensaje con el nombre del archivo y su tipo.

¡Listo! Ahora cada archivo que subas se procesará automáticamente.


### 📂 Código Adjunto
Puedes encontrar el template completo (bucket + rol + permisos + funciones Python) aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/17-lambda-s3-event-notifications.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="lD6TuB4khGw" provider="youtube" %}

---

### 💡 Próximos pasos
- AWS Lambda + Amazon API Gateway: Crea tu primera API REST en minutos
(Exponer tu función como API)
