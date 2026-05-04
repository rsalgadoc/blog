---
title: "Día 28: DynamoDB Streams: Reacciona a los cambios en tiempo real"
date: 2026-05-06
tags:
  - DynamoDB
  - Lambda
  - EventDriven
  - Serverless
toc: true
---

# 🌊 Día 28: ¡Tus datos tienen vida! Dispara acciones con Streams
¿Qué pasaría si pudieras enviar un correo de bienvenida automáticamente cuando un usuario se registra? ¿O actualizar un contador en otra tabla cuando se realiza una venta? En lugar de que tu aplicación tenga que hacer estas dos tareas a la vez, puedes usar **DynamoDB Streams**. Este servicio captura cada cambio en tu tabla y permite que otras herramientas (como AWS Lambda) reaccionen al instante.

### ¿Qué es un DynamoDB Stream?

*   **Registro de cambios**: Es un historial ordenado de lo que ha pasado en tu tabla (inserciones, modificaciones y borrados).
*   **Tiempo real**: Las acciones ocurren milisegundos después de que el dato cambió.
*   **Arquitectura desacoplada**: Tu base de datos no tiene que saber qué hace la Lambda; ella solo emite el evento y quien quiera escucharlo, que actúe.

### 🛠️ El Código (CloudFormation)
Vamos a crear una tabla que tenga el **Stream activado**. Para que funcione, debemos indicarle a DynamoDB qué tipo de información queremos que envíe cuando algo cambie.

#### 1. Tabla con Stream Habilitado

```yaml
TablaConEventos:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "RegistroActividad"
    AttributeDefinitions:
      - AttributeName: "Id"
        AttributeType: "S"
    KeySchema:
      - AttributeName: "Id"
        KeyType: "HASH"
    BillingMode: PAY_PER_REQUEST
    # ACTIVACIÓN DEL STREAM
    StreamSpecification:
      StreamViewType: NEW_AND_OLD_IMAGES # Envía el dato antes y después del cambio
```

### 📚 Conceptos Nuevos Explicados

#### 1. StreamViewType
Define qué información se incluirá en el evento del Stream:
*   **KEYS_ONLY**: Solo las llaves del elemento modificado.
*   **NEW_IMAGE**: El elemento completo tal como quedó después del cambio.
*   **OLD_IMAGE**: El elemento tal como estaba antes del cambio.
*   **NEW_AND_OLD_IMAGES**: Ambos (ideal para comparar qué cambió exactamente).

#### 2. Disparador (Trigger)
Es la conexión entre el Stream y una función **AWS Lambda**. Cuando llega un nuevo registro al Stream, AWS invoca a tu Lambda pasando los datos del cambio como un evento JSON.

#### 3. Retención de datos
Los eventos en el Stream solo viven por **24 horas**. Si tu función Lambda falla o no los procesa a tiempo, esos eventos desaparecerán.

### 🚀 Cómo Desplegarlo

Activa el flujo de eventos en tu infraestructura:

```shell
aws cloudformation deploy \
  --stack-name Dynamo-Streams-D28 \
  --template-file 28-dynamo-streams.yaml
```

### 📈 Cómo probar la "reacción" de tus datos

1. Ve a la consola de **DynamoDB** → **Explore items** → Tabla `RegistroActividad`.
2. Ve a la pestaña **Exports and streams**.
3. Verás que el Stream está "Enabled". Allí mismo podrías hacer clic en **Create trigger** para conectarlo a una Lambda existente.
4. Si creas un Item nuevo, DynamoDB generará un evento invisible que viajará por el Stream. En una arquitectura real, tu Lambda recibiría un JSON parecido a este:
   ```json
   "eventName": "INSERT",
   "dynamodb": {
       "NewImage": { "Id": {"S": "123"}, "Status": {"S": "Activo"} }
   }
   ```

### 📂 Código Adjunto

Puedes bajar el template con la especificación del Stream aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/28-dynamo-streams.yaml)

---

### 🎥 Video Tutorial

En el video de hoy construimos una automatización completa: insertamos un dato en DynamoDB y vemos cómo llega un email automáticamente usando Lambda y SES:

{% include video.html id="VIDEO_ID_DYNAMO_STREAMS" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 29: TTL (Time to Live) y copias de seguridad**: Aprenderemos a hacer que los datos se borren solos cuando ya no los necesitemos y cómo proteger nuestra tabla contra desastres.
