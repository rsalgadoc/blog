---
title: "Día 29: TTL y Backups: Limpieza automática y protección de datos"
date: 2026-05-07
tags:
  - DynamoDB
  - Seguridad
  - Backup
  - FinOps
toc: true
---

# ⏳ Día 29: Datos que se autodestruyen y copias de seguridad
¡Llegamos al final de nuestra serie de DynamoDB! Ya sabemos crear tablas rápidas y reactivas, pero un buen arquitecto también sabe cuándo deshacerse de los datos que ya no sirven y cómo protegerse ante un desastre. Hoy aprenderás el **TTL (Time to Live)** para ahorrar espacio y los métodos de **Backup** para dormir tranquilo.

### 1. TTL (Time to Live): La papelera automática
¿Para qué pagar por guardar logs de hace 3 años o sesiones de usuario que ya expiraron? 
*   **Ahorro total**: AWS borra los items expirados automáticamente y **no te cobra** por esas escrituras de borrado.
*   **Cómo funciona**: Eliges un atributo que contenga una fecha en formato *Unix Timestamp*. Cuando llega esa hora, DynamoDB marca el item para eliminarlo.

### 2. Backups y Point-in-Time Recovery (PITR)
*   **On-Demand Backups**: Fotos completas de tu tabla que puedes guardar por años.
*   **PITR (Point-in-Time Recovery)**: Es como una máquina del tiempo. Si lo activas, puedes restaurar tu tabla a **cualquier segundo exacto** de los últimos 35 días. Ideal si un script mal programado borra media base de datos por error.

### 🛠️ El Código (CloudFormation)
Vamos a crear una tabla de "Sesiones" que tenga el PITR activado para seguridad y el TTL configurado para limpiar sesiones viejas.

#### 1. Tabla con Auto-limpieza y Protección

```yaml
TablaSesionesSegura:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "SesionesApp"
    AttributeDefinitions:
      - AttributeName: "SessionId"
        AttributeType: "S"
    KeySchema:
      - AttributeName: "SessionId"
        KeyType: "HASH"
    BillingMode: PAY_PER_REQUEST
    # 1. ACTIVAR EL TTL (Auto-borrado)
    TimeToLiveSpecification:
      AttributeName: "ExpiraEn" # Nombre del campo con la fecha
      Enabled: true
    # 2. ACTIVAR LA MÁQUINA DEL TIEMPO (PITR)
    PointInTimeRecoverySpecification:
      PointInTimeRecoveryEnabled: true
```

### 📚 Conceptos Nuevos Explicados

#### 1. Unix Timestamp
Para que el TTL funcione, la fecha debe estar en segundos desde el 1 de enero de 1970 (ej: `1714834800`). Si lo pones en formato normal (YYYY-MM-DD), DynamoDB lo ignorará.

#### 2. Borrado Asíncrono
El TTL no borra el archivo exactamente al segundo que expira. Suele tardar entre unos minutos y 48 horas en desaparecer físicamente, pero para tu aplicación es como si ya no existiera.

#### 3. Restauración de Tablas
Cuando restauras un Backup o usas PITR, AWS crea una **tabla nueva**. No sobreescribe la actual. Esto te permite comparar datos antes de decidir cuál conservar.

### 🚀 Cómo Desplegarlo

Protege tus datos y automatiza su limpieza con un solo comando:

```shell
aws cloudformation deploy \
  --stack-name Dynamo-Final-D29 \
  --template-file 29-dynamo-final.yaml
```

### 📈 Cómo probar el TTL

1. Ve a la consola de **DynamoDB** → Tabla `SesionesApp`.
2. En la pestaña **Additional settings**, verifica que el TTL está "Enabled" apuntando al campo `ExpiraEn`.
3. Crea un Item y en el campo `ExpiraEn` pon un número de hace 5 minutos (puedes buscar "Unix Timestamp converter" en Google).
4. En un rato (puede tardar), verás que el registro desaparece solo. ¡Magia serverless!

### 📂 Código Adjunto

Puedes bajar el template final con seguridad y TTL aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/29-dynamo-final.yaml)

---

### 🎥 Video Tutorial

En el video de hoy te enseño cómo recuperar una tabla que fue borrada "accidentalmente" en menos de 5 minutos usando PITR:

{% include video.html id="VIDEO_ID_DYNAMO_BACKUP1" provider="youtube" %}

---

### 💡 ¿Qué sigue?
Hemos terminado con las bases de datos NoSQL. Ahora que tenemos dónde guardar archivos (S3) y dónde guardar datos (DynamoDB), es momento de aprender a enviar mensajes entre ellos. ¡Próximo módulo: **Amazon SNS y SQS** para desacoplar nuestra arquitectura!
