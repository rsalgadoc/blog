---
title: "Día 27: Índices en DynamoDB: GSI vs LSI"
date: 2026-05-05
tags:
  - DynamoDB
  - NoSQL
  - DatabaseDesign
  - Performance
toc: true
---

# 🔍 Día 27: ¿Cómo buscar datos sin usar la Llave Principal?
Hasta ahora, hemos aprendido que para ser rápidos en DynamoDB debemos buscar por la **Partition Key**. Pero, ¿qué pasa si en nuestra tabla de usuarios queremos buscar por `Email` en lugar de `UserId`? Si hiciéramos un *Scan*, sería lento y costoso. Para solucionar esto existen los **Índices**, que son como "copias" de nuestra tabla organizadas de otra forma.

### 1. GSI (Global Secondary Index) - El más flexible
Es el tipo de índice más potente y el que usarás el 90% del tiempo. 
*   **Flexibilidad**: Puedes elegir cualquier atributo como nueva Partition Key y Sort Key.
*   **Escalabilidad**: Tiene su propia capacidad (RCU/WCU) independiente de la tabla principal.
*   **Alcance**: Se puede consultar a través de toda la tabla, sin importar la partición original.

### 2. LSI (Local Secondary Index) - El hermano estricto
Es mucho más limitado y solo se puede crear al momento de crear la tabla.
*   **Restricción**: Debe compartir la **misma Partition Key** que la tabla original, pero usa una Sort Key distinta.
*   **Consistencia**: Permite lecturas "Fuertemente Consistentes" (cosa que el GSI no puede).

### 🛠️ El Código (CloudFormation)
Vamos a crear una tabla de usuarios donde la Partition Key es `UserId`, pero añadiremos un **GSI** para poder buscar instantáneamente por `Email`.

#### 1. Tabla con Índice Secundario Global (GSI)

```yaml
TablaUsuariosConGSI:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "UsuariosSeguridad"
    AttributeDefinitions:
      - AttributeName: "UserId"
        AttributeType: "S"
      - AttributeName: "Email"
        AttributeType: "S"
    KeySchema:
      - AttributeName: "UserId"
        KeyType: "HASH"
    BillingMode: PAY_PER_REQUEST
    # Definición del Índice
    GlobalSecondaryIndexes:
      - IndexName: "EmailIndex"
        KeySchema:
          - AttributeName: "Email"
            KeyType: "HASH"
        Projection:
          ProjectionType: "ALL" # Copia todos los atributos al índice
```

### 📚 Conceptos Nuevos Explicados

#### 1. Proyección (Projection)
Cuando creas un índice, debes decidir qué datos se "copian" de la tabla original al índice:
*   **KEYS_ONLY**: Solo las llaves (ahorra espacio).
*   **INCLUDE**: Solo los atributos que tú elijas.
*   **ALL**: Todos los atributos (más fácil de usar, pero más caro en almacenamiento).

#### 2. Replicación Asíncrona
Cuando escribes en la tabla principal, DynamoDB copia el dato al **GSI** de forma automática en milisegundos. Esta copia es **asíncrona**, por lo que los GSI solo soportan consistencia eventual.

#### 3. Diferencia Crítica: Creación
*   **GSI**: Puedes crearlo, modificarlo o borrarlo en cualquier momento, incluso si la tabla ya tiene datos.
*   **LSI**: **Solo** se puede crear cuando creas la tabla por primera vez. Si lo olvidas, tendrías que borrar y recrear la tabla.

### 🚀 Cómo Desplegarlo

Despliega tu tabla con capacidad de búsqueda avanzada:

```shell
aws cloudformation deploy \
  --stack-name Dynamo-Indices-D27 \
  --template-file 27-dynamo-indices.yaml
```

### 📈 Cómo probar tus índices

1. Ve a la consola de **DynamoDB** → **Explore items** → Tabla `UsuariosSeguridad`.
2. Crea un item con un `UserId` y un `Email`.
3. Ve a la pestaña **Query**. 
4. En el menú desplegable de "Source", cambia de [Table] a **[Index: EmailIndex]**.
5. Ahora puedes buscar directamente por el correo electrónico. ¡Es igual de rápido que buscar por el ID!

### 📂 Código Adjunto

Puedes bajar el template con la configuración del GSI aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/27-dynamo-indices.yaml)

---

### 🎥 Video Tutorial

En el video de hoy te explico cuándo vale la pena pagar por un GSI y cómo evitar el error común de crear demasiados índices:

{% include video.html id="VIDEO_ID_DYNAMO_INDEXES" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 28: DynamoDB Streams**: ¿Quieres que algo pase automáticamente cuando un dato cambia? Aprenderemos a disparar funciones Lambda al detectar inserciones o borrados.
