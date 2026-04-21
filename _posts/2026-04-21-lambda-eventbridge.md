---
title: "Día 13: Programando ejecuciones automáticas con Amazon EventBridge (Cron Jobs en Lambda)"
date: 2026-04-21
tags:
  - CloudFormation
  - Lambda
  - EventBridge
  - Python
toc: true
---

# ⏰ / ⏱️ Día 13: Ejecuta código automáticamente con EventBridge — Cron Jobs en Lambda

Hoy vamos a aprender cómo hacer que nuestra función Lambda se ejecute **automáticamente según un horario**, como si fuera un cron job tradicional, pero de forma serverless.

### ¿Por qué usar EventBridge con Lambda?

* Ejecutar tareas programadas (reportes diarios, limpieza, sincronizaciones, etc.).
* Reemplazar servidores con cron de forma más económica y escalable.
* Totalmente serverless: solo pagas cuando se ejecuta.
* Muy flexible en los horarios.

### 🛠️ El Código (CloudFormation)

```yaml
  # Programación para pruebas (cada 5 minutos)
  TestEvery5MinutesSchedule:
    Type: AWS::Scheduler::Schedule
    Properties:
      Name: Test-Lambda-Every5Minutes
      Description: Ejecuta cada 5 minutos usando el nuevo Scheduler
      ScheduleExpression: "rate(5 minutes)"
      State: ENABLED
      FlexibleTimeWindow:
        Mode: "OFF" # Ejecución exacta. Puedes poner "FLEXIBLE" para optimizar costos.
      Target:
        Arn: !GetAtt MyScheduledLambda.Arn
        RoleArn: !GetAtt SchedulerToLambdaRole.Arn

  # Programación diaria (Producción)
  DailySchedule:
    Type: AWS::Scheduler::Schedule
    Properties:
      Name: Daily-Lambda-At8AM
      Description: Ejecuta a las 8:00 AM usando zona horaria específica
      ScheduleExpression: "cron(0 8 * * ? *)"
      ScheduleExpressionTimezone: "UTC" # ¡Ahora puedes poner "America/Santiago" o similar!
      State: DISABLED
      FlexibleTimeWindow:
        Mode: "OFF"
      Target:
        Arn: !GetAtt MyScheduledLambda.Arn
        RoleArn: !GetAtt SchedulerToLambdaRole.Arn
```

### 📚 Conceptos Nuevos Explicados
#### 1. Amazon EventBridge Scheduler
Servicio que permite programar la ejecución de Lambdas (y otros servicios).
#### 2. ScheduleExpression

* `rate(5 minutes)` → Se ejecuta cada 5 minutos (ideal para pruebas)
* `cron(0 8 * * ? *)` → Cron clásico: todos los días a las 8:00 AM UTC

#### 3. State: ENABLED / DISABLED
Puedes activar o desactivar los Schedule según necesites.
#### 4. Modificar en caliente
Puedes editar los Schedule directamente desde la consola de EventBridge sin tener que redeployar todo el stack.
### 🚀 Cómo Desplegarlo y Probarlo

```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://21-lambda-eventbridge.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```
### 📈 Cómo probar fácilmente

1. Despliega el template.
2. Ve a **EventBridge → Schedules**.
3. Busca el Schedule `Test-Lambda-Every5Minutes`.
4. Espera máximo 5 minutos y revisa los logs de tu Lambda.
5. **Para probar más rápido**:
  * Edita el Schedule → cambia `rate(5 minutes)` a `rate(1 minute)`.
  * Guarda los cambios.
  * Espera 1 minuto y revisa los logs.


**Consejo**: Durante el desarrollo, mantén el Schedule de 5 minutos activada y la diaria desactivada. Cuando termines las pruebas, desactiva la de prueba y activa la diaria.

### 📂 Código Adjunto
Puedes encontrar el template completo aquí:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/21-lambda-eventbridge.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="CFMbneNhl4s" provider="youtube" %}

---

### 💡 Próximos pasos
AWS Lambda + DynamoDB: Crea una API simple para guardar y leer datos (Primer contacto con base de datos serverless)
