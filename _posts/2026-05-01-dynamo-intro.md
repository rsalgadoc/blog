---
title: "Día 23: ¿Qué es DynamoDB y cuándo usar NoSQL?"
date: 2026-05-01
tags:
  - DynamoDB
  - NoSQL
  - Serverless
  - Databases
toc: true
---

# ⚡ Día 23: Bases de datos a la velocidad del rayo con DynamoDB
Después de dominar S3, hoy entramos en el corazón de las aplicaciones modernas: las bases de datos. Pero no vamos a hablar de las clásicas tablas con filas y columnas de toda la vida. Hoy conocerás **Amazon DynamoDB**, la base de datos NoSQL de AWS que puede manejar millones de peticiones por segundo con una latencia de milisegundos.

### ¿Qué hace a DynamoDB tan especial?

*   **Serverless total**: No eliges cuánta RAM o CPU necesitas; AWS se encarga de todo el escalado.
*   **Velocidad consistente**: No importa si tienes 1 GB o 100 TB de datos, el tiempo de respuesta siempre es de un solo dígito de milisegundo.
*   **Disponibilidad extrema**: Tus datos se replican automáticamente en tres zonas de disponibilidad (AZ) distintas.
*   **Sin esquemas rígidos**: Cada registro puede tener atributos diferentes sin necesidad de alterar una tabla.

### 🤔 ¿Cuándo elegir NoSQL (DynamoDB) en lugar de SQL?


| Situación | ¿Usar DynamoDB? |
| :--- | :--- |
| Necesitas escalar a millones de usuarios | ✅ **Sí** |
| Datos con estructura flexible | ✅ **Sí** |
| Relaciones complejas entre tablas (Joins) | ❌ No (Mejor usar RDS) |
| Aplicaciones de baja latencia (juegos, carritos de compra) | ✅ **Sí** |
| Consultas de analítica profunda y reportes complejos | ❌ No |

### 🛠️ El Código (CloudFormation)
Para empezar, vamos a crear nuestra primera tabla. En DynamoDB, lo único que debemos definir obligatoriamente al inicio es el nombre de la tabla y su llave principal.

#### 1. Mi primera tabla de DynamoDB

```yaml
MiTablaUsuarios:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "UsuariosApp"
    # Definimos el esquema de la llave (Partition Key)
    AttributeDefinitions:
      - AttributeName: "UserId"
        AttributeType: "S" # S de String
    KeySchema:
      - AttributeName: "UserId"
        KeyType: "HASH" # HASH es el identificador único
    # Modo de pago por uso (ideal para empezar)
    BillingMode: PAY_PER_REQUEST
    Tags:
      - Key: "Modulo"
        Value: "DynamoDB-Intro"
```

### 📚 Conceptos Nuevos Explicados

#### 1. NoSQL (Not Only SQL)
A diferencia de SQL, donde todo debe encajar en columnas fijas, en NoSQL priorizamos el rendimiento y la flexibilidad. No hay "JOINs" entre tablas; los datos se guardan de forma que leerlos sea lo más rápido posible.

#### 2. Partition Key (Llave de Partición)
Es el identificador único de cada registro. AWS usa este valor para saber en qué parte física de sus servidores guardar tu dato.

#### 3. Billing Mode: PAY_PER_REQUEST
Es el modo "On-Demand". No pagas nada por hora; solo pagas por cada lectura o escritura real que hagas. Es perfecto para el blog porque si nadie entra a tu app, te cuesta **$0**.

### 🚀 Cómo Desplegarlo

Crea tu primera base de datos NoSQL con este comando:

```shell
aws cloudformation deploy \
  --stack-name Dynamo-Intro-D23 \
  --template-file 01-dynamo-intro.yaml
```

### 📈 Cómo probar tu tabla

1. Ve a la consola de **DynamoDB** → **Tables** → Selecciona `UsuariosApp`.
2. Haz clic en **Explore table items**.
3. Haz clic en **Create item**.
4. En `UserId`, pon `101`.
5. Haz clic en **Add new attribute** → **String** → Nombre: `Nombre`, Valor: `Tu Nombre`.
6. ¡Guárdalo! Acabas de insertar tu primer registro en una base de datos de clase mundial.

### 📂 Código Adjunto

Puedes bajar el template de esta primera tabla aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/05/01-dynamo-intro.yaml)

---

### 🎥 Video Tutorial

Mira el proceso paso a paso en video:

{% include video.html id="z7SzhrvpzzU" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 24: Conceptos clave: Tablas, Items y Atributos**: Mañana aprenderemos la anatomía real de los datos y cómo se diferencian de una base de datos tradicional.
