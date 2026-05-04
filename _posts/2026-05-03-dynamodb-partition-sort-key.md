---
title: "Día 25: El secreto del éxito: Partition Key y Sort Key"
date: 2026-05-03
tags:
  - DynamoDB
  - NoSQL
  - DatabaseDesign
  - Architecture
toc: true
---

# 🔑 Día 25: Las llaves que abren la velocidad de milisegundos
Si intentas usar DynamoDB como si fuera SQL, vas a sufrir. En SQL puedes buscar por cualquier columna, pero en DynamoDB, la forma en que recuperas tus datos depende casi totalmente de cómo diseñes tu **Primary Key**. Hoy entenderás la diferencia entre una llave simple y una compuesta.

### 1. Partition Key (PK) - La brújula de AWS
También conocida como **Hash Key**. Es obligatoria. AWS pasa este valor por un algoritmo (hash) para decidir en qué servidor físico guardará tus datos. 
*   **Regla de oro**: Debe estar bien distribuida. Si todos tus usuarios tienen la misma PK, crearás un "Hot Partition" y tu base de datos se volverá lenta.

### 2. Sort Key (SK) - El orden del caos
También conocida como **Range Key**. Es opcional. Permite guardar varios elementos con la misma *Partition Key*, pero organizados por la *Sort Key*.
*   **Su superpoder**: Te permite hacer consultas como "tráeme todos los pedidos del Usuario X (PK) que ocurrieron en el último mes (SK)".

### 🛠️ El Código (CloudFormation)
Vamos a crear una tabla de "Pedidos" (Orders) que utiliza una **Llave Compuesta** (PK + SK).

#### 1. Tabla de Pedidos con Llave Compuesta

```yaml
TablaPedidos:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "PedidosClientes"
    AttributeDefinitions:
      - AttributeName: "CustomerId" # Partition Key
        AttributeType: "S"
      - AttributeName: "OrderId"    # Sort Key
        AttributeType: "S"
    KeySchema:
      - AttributeName: "CustomerId"
        KeyType: "HASH" # PK
      - AttributeName: "OrderId"
        KeyType: "RANGE" # SK
    BillingMode: PAY_PER_REQUEST
```

### 📚 Conceptos Nuevos Explicados

#### 1. Llave Primaria Simple
Solo tiene **Partition Key**. El valor debe ser único en toda la tabla (como el DNI de una persona).

#### 2. Llave Primaria Compuesta (PK + SK)
La combinación de ambas debe ser única. Esto permite modelar relaciones 1:N (un cliente, muchos pedidos).
*   **Ejemplo**: 
    *   PK: `USER#123` | SK: `ORDER#001`
    *   PK: `USER#123` | SK: `ORDER#002`

#### 3. Query vs Scan
*   **Query**: Buscas por PK. Es extremadamente rápido y barato porque AWS sabe exactamente a qué servidor ir.
*   **Scan**: Buscas en toda la tabla (como leer un libro completo para encontrar una palabra). Es lento y muy caro. **¡Evítalo a toda costa!**

### 🚀 Cómo Desplegarlo

Crea la tabla de pedidos con este comando:

```shell
aws cloudformation deploy \
  --stack-name Dynamo-Keys-D25 \
  --template-file 03-dynamodb-partition-sort-key.yaml
```

### 📈 Cómo probar las llaves

1. Ve a la consola de **DynamoDB** → **Explore items** → Tabla `PedidosClientes`.
2. Crea varios items usando el mismo `CustomerId` (ej. `CLIENTE_A`) pero diferentes `OrderId`.
3. Ahora ve a la pestaña de "Query". Selecciona `CustomerId` e ingresa `CLIENTE_A`.
4. Verás cómo DynamoDB te devuelve instantáneamente todos los pedidos de ese cliente específico sin revisar el resto de la tabla.

### 📂 Código Adjunto

Puedes bajar el template de la tabla con llaves compuestas aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/05/03-dynamodb-partition-sort-key.yaml)

---

### 🎥 Video Tutorial

Mira el proceso paso a paso en video:

{% include video.html id="yhjHWDMZqPQ" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 26: Lecturas y escrituras: Consistencia y capacidad (RCU/WCU)**: ¿Cómo cobra AWS por usar DynamoDB? Aprenderemos a medir el "esfuerzo" de nuestra base de datos.
