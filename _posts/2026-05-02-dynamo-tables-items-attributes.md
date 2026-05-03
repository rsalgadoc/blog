---
title: "Día 24: Anatomía de DynamoDB: Tablas, Items y Atributos"
date: 2026-05-02
tags:
  - DynamoDB
  - NoSQL
  - DatabaseDesign
  - Serverless
toc: true
---

# 📋 Día 24: ¿Cómo se organizan los datos en DynamoDB?
Ayer creamos nuestra primera tabla, pero hoy vamos a entender qué hay dentro. Si vienes del mundo SQL (MySQL, PostgreSQL), prepárate para desaprender un poco. En DynamoDB no hablamos de filas y columnas, sino de una jerarquía mucho más flexible y potente.

### 🧬 La jerarquía de datos

1.  **Tablas (Tables)**: Es el contenedor principal (similar a una tabla SQL).
2.  **Elementos (Items)**: Es un registro individual dentro de la tabla (similar a una "fila").
3.  **Atributos (Attributes)**: Son los datos específicos de cada elemento (similar a las "columnas").

### ¿Por qué decimos que es "Schema-less"?
En una base de datos tradicional, si una tabla tiene las columnas `Nombre` y `Email`, todas las filas DEBEN tener esas columnas. En DynamoDB, un **Item** puede tener `Nombre` y `Email`, pero el siguiente Item puede tener `Nombre` y `Twitter`, ¡y no pasa nada! La tabla no explota.

### 🛠️ El Código (CloudFormation)
Vamos a crear una tabla de "Productos" que nos permita ver cómo se definen los tipos de datos básicos.

#### 1. Tabla de Catálogo de Productos

```yaml
TablaProductos:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "CatalogoProductos"
    # Definimos solo la llave principal (obligatorio)
    AttributeDefinitions:
      - AttributeName: "SKU"
        AttributeType: "S" # S = String (Texto)
    KeySchema:
      - AttributeName: "SKU"
        KeyType: "HASH"
    BillingMode: PAY_PER_REQUEST
```

### 📚 Conceptos Nuevos Explicados

#### 1. Atributos Escalares (Tipos Básicos)
*   **S (String)**: Texto.
*   **N (Number)**: Números (enteros, decimales).
*   **B (Binary)**: Datos binarios (como imágenes en miniatura).

#### 2. Atributos de Documento (Tipos Complejos)
Aquí es donde DynamoDB brilla. Puedes guardar:
*   **L (List)**: Una lista ordenada de valores (como una lista de tags: `['electronica', 'oferta']`).
*   **M (Map)**: Un objeto JSON anidado dentro de un atributo. ¡Puedes tener mini-estructuras dentro de un Item!

#### 3. El tamaño del Item
Ojo: Un solo **Item** (con todos sus atributos) no puede pesar más de **400 KB**. DynamoDB está hecho para datos rápidos y ligeros, no para guardar archivos pesados (para eso usamos S3).

### 🚀 Cómo Desplegarlo

Actualiza tu infraestructura para añadir la tabla de productos:

```shell
aws cloudformation deploy \
  --stack-name Dynamo-Conceptos-D24 \
  --template-file 02-dynamo-tables-items-attributes.yaml
```

### 📈 Cómo probar la flexibilidad

1. Ve a la consola de **DynamoDB** → **Explore items** → Tabla `CatalogoProductos`.
2. Crea un Item: `SKU: PROD-001`, `Nombre: Camiseta`.
3. Crea otro Item: `SKU: PROD-002`, `Nombre: Laptop`, y añade un nuevo atributo tipo **Number** llamado `Precio: 1200`.
4. Observa cómo la tabla muestra ambos, aunque uno tenga precio y el otro no. ¡Eso es flexibilidad NoSQL!

### 📂 Código Adjunto

Puedes bajar el template de la tabla de productos aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/05/02-dynamo-tables-items-attributes.yaml)

---

### 🎥 Video Tutorial

En el video de hoy comparamos una tabla de Excel con una tabla de DynamoDB para que veas visualmente la diferencia entre filas y elementos:

{% include video.html id="VIDEO_ID_DYNAMO_ANATOMY" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 25: La importancia de la Partition Key y Sort Key**: Mañana veremos el concepto más importante de DynamoDB. Si entiendes las llaves, dominas la base de datos.
