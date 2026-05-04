---
title: "Día 26: El motor de DynamoDB: Consistencia y Capacidad (RCU/WCU)"
date: 2026-05-04
tags:
  - DynamoDB
  - Performance
  - CostOptimization
  - Architecture
toc: true
---

# ⏱️ Día 26: ¿Cuánta potencia necesita tu base de datos?
En las bases de datos tradicionales, hablamos de RAM y CPU. En DynamoDB, hablamos de **Capacidad de Lectura y Escritura**. Hoy aprenderás a medir el "esfuerzo" que hace tu tabla para procesar datos y cómo elegir entre pagar por uso o reservar potencia para ahorrar dinero.

### 1. Las unidades de medida: RCU y WCU
AWS mide el rendimiento de tu tabla con estas dos unidades:

*   **WCU (Write Capacity Unit)**: 1 WCU permite escribir 1 KB por segundo.
*   **RCU (Read Capacity Unit)**: 1 RCU permite leer 4 KB por segundo (en lectura eventualmente consistente).

### 2. ¿Lectura Fuerte o Eventual?
Aquí es donde muchos se confunden. Cuando lees datos en DynamoDB, tienes dos opciones:
*   **Eventualmente Consistente (Por defecto)**: Es más rápida y barata. Puede que leas un dato un milisegundo antes de que se actualice en todas las copias de AWS. (Gasta 0.5 RCU por 4KB).
*   **Fuertemente Consistente**: Te asegura que leerás el dato más reciente, pero cuesta el doble. (Gasta 1 RCU por 4KB).

### 🛠️ El Código (CloudFormation)
Hasta ahora hemos usado `PAY_PER_REQUEST`. Hoy veremos cómo configurar una tabla con **Capacidad Provisionada**, ideal cuando ya sabes cuánto tráfico tendrá tu app y quieres predecir tus costos.

#### 1. Tabla con Capacidad Definida

```yaml
TablaConCapacidad:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: "InventarioGlobal"
    AttributeDefinitions:
      - AttributeName: "ItemId"
        AttributeType: "S"
    KeySchema:
      - AttributeName: "ItemId"
        KeyType: "HASH"
    # Configuramos la potencia reservada
    BillingMode: PROVISIONED
    ProvisionedThroughput:
      ReadCapacityUnits: 5  # Capacidad de lectura
      WriteCapacityUnits: 5 # Capacidad de escritura
```

### 📚 Conceptos Nuevos Explicados

#### 1. Provisioned Mode vs. On-Demand
*   **On-Demand (Pay-per-request)**: Pagas por cada lectura/escritura real. Ideal para tráfico impredecible o apps que están empezando.
*   **Provisioned**: Tú dices "quiero 5 RCU y 5 WCU". AWS te cobra un monto fijo por hora. Es más barato si tienes un tráfico constante y alto.

#### 2. Throttling (Estrangulamiento)
Si configuraste 5 WCU pero intentas escribir 10 KB por segundo, AWS lanzará un error de `ProvisionedThroughputExceededException`. Tu aplicación "chocará" contra el muro que tú mismo definiste para ahorrar.

#### 3. Consistencia Eventual
Recuerda que DynamoDB guarda tus datos en 3 lugares distintos. La consistencia eventual lee de la copia más cercana, aunque la actualización de las otras dos esté en proceso (tarda milisegundos).

### 🚀 Cómo Desplegarlo

Crea la tabla con capacidad controlada:

```shell
aws cloudformation deploy \
  --stack-name Dynamo-Capacidad-D26 \
  --template-file 26-dynamo-capacidad.yaml
```

### 📈 Cómo probar la consistencia

1. Cuando uses el SDK (Python/Node.js) para leer un dato, busca el parámetro `ConsistentRead`.
2. Si lo pones en `True`, estarás forzando una **Lectura Fuerte** (más lenta y cara).
3. Si lo dejas en `False`, estarás usando **Consistencia Eventual** (más eficiente).

### 📂 Código Adjunto

Puedes bajar el template con capacidad provisionada aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/26-dynamo-performance.yaml)

---

### 🎥 Video Tutorial

En el video de hoy te enseño a usar la calculadora de AWS para saber cuántos RCU y WCU necesitas realmente para tu aplicación:

{% include video.html id="VIDEO_ID_DYNAMO_CAPACITY" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 27: Índices: GSI (Global) vs LSI (Local)**: ¿Qué pasa si necesitas buscar por algo que no es tu Partition Key? Aprenderemos a crear "vistas" de nuestros datos.
