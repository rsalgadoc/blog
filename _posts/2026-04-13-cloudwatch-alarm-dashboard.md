---
title: "Día 5: Observabilidad - Alarmas y Dashboards con CloudWatch"
date: 2026-04-13
tags:
  - AWS
  - CloudFormation
  - CloudWatch
  - Alarm
  - Dashboard
toc: true
---

# 👁️ Día 5: CloudWatch - Tu Torre de Control en AWS
Tu servidor ya está corriendo y tiene contenido, pero ¿qué pasa si se queda sin memoria o el CPU llega al 100%? Hoy aprenderemos a configurar CloudWatch usando CloudFormation para crear Alarmas que nos avisen de problemas y un Dashboard para visualizar la salud de nuestra EC2 en tiempo real.

### ¿Por qué es importante esto?
No puedes gestionar lo que no puedes medir. Implementar observabilidad te permite:
Reaccionar rápido: Enterarte de una caída antes que tus usuarios.
Optimizar costos: Ver si tu instancia es demasiado grande (o pequeña) para el tráfico que recibe.
Histórico de datos: Analizar patrones de uso durante el día o la semana.
Automatización: Disparar acciones automáticas (como reiniciar la instancia) si algo falla.

### 🛠️ El Código (CloudFormation)
En este post nos enfocamos en los recursos de monitoreo que se conectan a tu instancia actual.
#### 1. La Alarma de CPU
Agregamos una alerta que vigila si nuestro servidor está bajo mucho estrés:

```yaml
AlarmaCPUAlta:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmDescription: "Alerta si el CPU supera el 45% por más de 2 minutos"
    MetricName: CPUUtilization
    Namespace: AWS/EC2
    Statistic: Average
    Period: 60 # Segundos
    EvaluationPeriods: 2
    Threshold: 45
    ComparisonOperator: GreaterThanThreshold
    Dimensions:
      - Name: InstanceId
        Value: !Ref MiServidorWeb # Referencia a tu EC2
```

#### 2. El Dashboard Visual
Creamos un tablero con gráficos para ver el CPU y la Red de un vistazo:
```yaml
MiDashboard:
  Type: AWS::CloudWatch::Dashboard
  Properties:
    DashboardName: !Sub "Monitor-EC2-${AWS::StackName}"
    DashboardBody: !Sub |
      {
        "widgets": [
          {
            "type": "metric",
            "properties": {
              "metrics": [ [ "AWS/EC2", "CPUUtilization", "InstanceId", "${MiServidorWeb}" ] ],
              "title": "Uso de CPU (%)",
              "region": "${AWS::Region}"
            }
          }
        ]
      }
```

### 📚 Conceptos Nuevos Explicados
#### 1. Métricas (Metrics)
Son los datos numéricos que AWS recolecta de tus recursos. Por defecto, EC2 envía métricas cada 5 minutos (monitoreo básico) sin costo adicional.
#### 2. Alarmas y Umbrales (Thresholds)
Una alarma vigila una métrica. Si la métrica cruza el Threshold (umbral) durante los EvaluationPeriods (periodos de evaluación) definidos, la alarma cambia de estado a ALARM.
#### 3. Namespace y Dimensions
Namespace: Es el contenedor de la métrica (ej. AWS/EC2).
Dimensions: Es el filtro para saber exactamente qué recurso medir. Usamos InstanceId para medir solo nuestra instancia específica.
#### 4. Dashboards como Código
Los tableros de CloudWatch se definen internamente como un JSON. CloudFormation nos permite "dibujar" estos tableros definiendo la posición y el tipo de gráfico (widget).

### 🚀 Cómo Desplegarlo
Actualiza tu stack usando la CLI o la consola de AWS:
```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://dia7-cloudwatch.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=tu-llave
```

### 📈 Cómo probar tu Alarma
Para ver la alarma en acción sin esperar a tener tráfico real, puedes estresar el CPU de tu instancia:
Conecta por SSH a tu EC2.
Instala la herramienta de estrés: 
```shell
sudo yum install stress -y
```
Ejecuta: 
```shell
stress --cpu 1 --timeout 900 
```
Ve a la consola de CloudWatch > Alarms y verás cómo cambia a color rojo.

### 📂 Código Adjunto
Puedes encontrar el template completo con la EC2 + Alarma + Dashboard aquí:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/13-cloudwatch-alarm-dashboard.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="Ouh9J_Pa2AM" provider="youtube" %}

---

### 💡 Próximos pasos
- Enviar una notificación por correo (SNS) cuando la alarma se active.
