---
title: "Día 8: Funciones serverless con AWS Lambda: tu primera función en Python/Node.js"
date: 2026-04-16
tags:
  - CloudFormation
  - Lambda
  - Python
  - Node
toc: true
---

# 🚀 Día 8: ¡Sin servidores! Ejecuta código sin preocuparte de EC2 ni nada
Hoy damos un salto enorme hacia el mundo **serverless**. Vamos a crear nuestra primera función con **AWS Lambda**, el servicio que te permite ejecutar código sin gestionar servidores, sin pagar cuando no se usa y que se escala automáticamente.
### ¿Por qué usar AWS Lambda?

* **Serverless real**: Olvídate de instancias, actualizaciones y parches.
* **Paga por uso**: Solo pagas por los milisegundos que tu código se ejecuta.
* **Fácil de integrar**: Se conecta con casi todo en AWS (S3, SNS, API Gateway, etc.).
* Ideal para tareas pequeñas, automatizaciones, backend de APIs o procesar eventos.


### 🛠️ El Código (CloudFormation)
Vamos a crear una función Lambda muy simple que devuelva un mensaje de saludo. Te muestro las versiones en **Python y Node.js** (elige la que prefieras).


#### 1. Rol de ejecución IAM (necesario para que Lambda pueda correr)

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

#### 2. La función Lambda - Versión Python

```yaml
MiPrimeraLambdaPython:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: HolaMundo-Python
    Runtime: python3.12
    Role: !GetAtt LambdaExecutionRole.Arn
    Handler: index.lambda_handler
    Timeout: 30
    Code:
      ZipFile: |
        def lambda_handler(event, context):
            print("¡Hola desde AWS Lambda en Python!")
            return {
                'statusCode': 200,
                'body': '¡Hola Mundo! Esta es mi primera función Lambda en Python 🚀'
            }
```
#### 3. La función Lambda - Versión Node.js (alternativa)

```yaml
MiPrimeraLambdaNode:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: HolaMundo-Node
    Runtime: nodejs20.x
    Role: !GetAtt LambdaExecutionRole.Arn
    Handler: index.handler
    Timeout: 30
    Code:
      ZipFile: |
        exports.handler = async (event) => {
            console.log("¡Hola desde AWS Lambda en Node.js!");
            return {
                statusCode: 200,
                body: "¡Hola Mundo! Esta es mi primera función Lambda en Node.js 🚀"
            };
        };
```

### 📚 Conceptos Nuevos Explicados
#### 1. AWS Lambda
Es el servicio serverless de cómputo de AWS. Tú solo subes el código y AWS se encarga del resto: ejecución, escalado, alta disponibilidad y facturación.
#### 2. Handler
Es la función que AWS llama cuando se invoca tu Lambda. En Python es archivo.función, en Node.js es archivo.handler.
#### 3. Runtime
El entorno de ejecución (python3.12, nodejs20.x, etc.). AWS mantiene estos entornos actualizados.
#### 4. Inline Code (ZipFile)
Para ejemplos simples podemos poner el código directamente en el template de CloudFormation. En proyectos reales normalmente subes un .zip a S3.


### 🚀 Cómo Desplegarlo
Actualiza tu stack de CloudFormation (o crea uno nuevo si prefieres):

```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://16-lambda-python-node.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 📈 Cómo probar tu primera Lambda

1. Ve a la consola de **Lambda** → busca tu función (HolaMundo-Python o HolaMundo-Node).
2. Haz clic en **Test → Create new test event** (puedes dejar el evento vacío).
3. Ejecuta el test.
4. Verás la respuesta con el mensaje “¡Hola Mundo!” y los logs en la sección de ejecución.

¡Listo! Acabas de ejecutar código serverless por primera vez.


### 📂 Código Adjunto
Puedes encontrar el template completo con el rol + función Lambda (Python y Node.js) aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/16-lambda-python-node.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="29dSGmm-Dck" provider="youtube" %}

---

### 💡 Próximos pasos
- 
