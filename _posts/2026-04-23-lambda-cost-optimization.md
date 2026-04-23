---
title: "Día 15: Optimizando costos y rendimiento en AWS Lambda: Memory, Timeout y Concurrency"
date: 2026-04-23
tags:
  - Lambda
  - Performance
  - Cost Optimization
  - CloudWatch
  - Serverless
toc: true
---

# ⚡ Día 15: Optimización en Lambda — ¡Más rápido y más barato!

Ayer aprendimos a persistir datos, pero hoy vamos a tocar las "perillas" de nuestra función. Mucha gente piensa que en Serverless solo despliegas y te olvidas, pero configurar mal la **Memoria**, el **Timeout** o la **Concurrencia** puede hacer que tu factura de AWS suba innecesariamente o que tu aplicación falle bajo carga.

Hoy aprenderás a tunear tu Lambda para encontrar el equilibrio perfecto entre rendimiento y costo.

### 🧠 1. Memoria (Memory)
En AWS Lambda, la memoria es el **único control** que tienes sobre el poder de cómputo. Al asignar más memoria, AWS asigna proporcionalmente más **CPU** y ancho de banda de red.

*   **El Mito:** "Menos memoria = menos costo". 
*   **La Realidad:** A veces, darle 1024MB a una función en lugar de 128MB hace que termine tan rápido que terminas pagando **menos** (el famoso *Power Tuning*).
*   **Best Practice:** No adivines. Usa herramientas como **AWS Lambda Power Tuning** para encontrar el "punto dulce" donde el costo y el tiempo de ejecución se cruzan de forma óptima.

### ⏳ 2. Tiempo de espera (Timeout)
El Timeout es el tiempo máximo que permites que tu función corra antes de que AWS la "mate". 

*   **Riesgo:** Si es muy corto, tus procesos pesados fallarán. Si es muy largo (ej. 15 min para una API), un error en tu código o una base de datos lenta podría mantener la Lambda encendida gastando dinero innecesariamente.
*   **Best Practice:** Configura el timeout a un **20-30% por encima** de tu tiempo de ejecución promedio. Si tu API tarda 200ms, un timeout de 3 segundos es más que suficiente.

### 🚦 3. Concurrencia (Concurrency)
La concurrencia es el número de instancias de tu función que se ejecutan al mismo tiempo.

*   **Reserved Concurrency:** Garantiza que tu función siempre tenga capacidad disponible, pero también actúa como un **límite máximo**. Úsalo para evitar que una Lambda consuma toda la capacidad de tu cuenta de AWS o para no saturar una base de datos tradicional (como RDS).
*   **Provisioned Concurrency:** ¡Adiós a los *Cold Starts*! Mantiene funciones "calientes" listas para responder instantáneamente. Ideal para APIs críticas donde la latencia inicial es inaceptable.

### 🛠️ Ejemplo de Configuración (CloudFormation)

Mira cómo definimos estos límites de forma profesional en un template:

```yaml
  LambdaOptimizada:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: Lambda-High-Performance
      Runtime: python3.12
      Handler: index.lambda_handler
      Role: !GetAtt LambdaExecutionRole.Arn
      # --- AJUSTES DE OPTIMIZACIÓN ---
      MemorySize: 512                 # Más memoria = más CPU
      Timeout: 10                     # Evita procesos zombis de larga duración
      ReservedConcurrentExecutions: 5 # No queremos saturar nuestra DB
      # -------------------------------
      Code:
        ZipFile: |
          import json
          def lambda_handler(event, context):
              return {'statusCode': 200, 'body': '¡Optimizada!'}
```

### 📊 Monitoreo: ¿Cómo saber si voy bien?

No puedes mejorar lo que no mides. En **Amazon CloudWatch Metrics**, debes vigilar estos tres indicadores:

1.  **Duration:** ¿Cuánto tarda realmente? Si estás muy cerca del Timeout, actúa.
2.  **Invocations vs Throttles:** Si tienes "Throttles", significa que llegaste al límite de concurrencia y AWS está rechazando peticiones.
3.  **CloudWatch Insights:** Usa esta query para ver cuánta memoria usas realmente:
    ```sql
    filter @type == "REPORT"
    | stats max(@maxMemoryUsed) as max_mem, avg(@duration) as avg_dur by @logStream
    ```

### 💡 Tips de Oro para ahorrar hoy mismo

1.  **Evita el "Fat Lambda":** No metas librerías gigantes que no usas. Menos código = menor tiempo de inicialización (Cold Start).
2.  **Conexiones persistentes:** Si usas bases de datos, inicializa el cliente **fuera** del `lambda_handler` para que se reutilice en futuras llamadas.
3.  **Log Level:** No imprimas todo en producción. Los logs en CloudWatch también cuestan. Usa variables de entorno para cambiar a `ERROR` en lugar de `DEBUG`.

---

### 📂 Recursos del día
- [AWS Lambda Power Tuning (GitHub)](https://github.com/alexcasalboni/aws-lambda-power-tuning)
- [Calculadora de Costos Serverless](https://calculator.aws)

---

### 📂 Código Adjunto
Puedes encontrar el template y el código de la función aquí:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/23-lambda-cost-optimization.yaml)

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="0jj7G41XbX4" provider="youtube" %}

---

### 💡 Próximo paso
- S3, Introducción a S3 y creación de tu primer Bucket.
