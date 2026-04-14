---
title: "Día 6: Notificaciones en tiempo real - Alertas por Email con SNS"
date: 2026-04-14
tags:
  - CloudFormation
  - CloudWatch
  - Alarm
  - SNS
toc: true
---

# 📩 Día 6: ¡No más sorpresas! Recibe Alertas en tu Email
Ayer aprendimos a crear una alarma que se pone en rojo cuando el CPU sube, pero ¿de qué sirve si no estás mirando la consola de AWS? Hoy vamos a conectar esa alarma con Amazon SNS (Simple Notification Service) para que AWS te envíe un correo electrónico automáticamente en cuanto algo falle.

### ¿Por qué usar SNS con tus Alarmas?
Monitoreo Pasivo: No necesitas refrescar el navegador; el problema llega a ti.
Escalabilidad: Puedes enviar el mismo aviso a todo tu equipo de TI simultáneamente.
Protocolos Variados: Aunque hoy usamos Email, SNS puede enviar SMS, disparar funciones Lambda o notificar a Slack.

### 🛠️ El Código (CloudFormation)
Para que esto funcione, necesitamos dos cosas: crear un Tópico (el canal de comunicación) y decirle a la Alarma que use ese canal.
#### 1. El Tópico SNS y la Suscripción
Aquí definimos a dónde queremos que llegue el aviso:

```yaml
MiTopicoAlertas:
  Type: AWS::SNS::Topic
  Properties:
    TopicName: "Alertas-Criticas-EC2"

SuscripcionEmail:
  Type: AWS::SNS::Subscription
  Properties:
    TopicArn: !Ref MiTopicoAlertas
    Endpoint: "tu-correo@ejemplo.com" # 👈 ¡Cambia esto por tu email real!
    Protocol: email

```

#### 2. Conectar la Alarma con SNS
Ahora modificamos la AlarmaCPUAlta que creamos ayer para agregar la propiedad AlarmActions:
```yaml
AlarmaCPUAlta:
  Type: AWS::CloudWatch::Alarm
  Properties:
    # ... (Mantén las propiedades de ayer: MetricName, Threshold, etc.)
    AlarmActions:
      - !Ref MiTopicoAlertas # 👈 Aquí le decimos: "Si te activas, avisa a este tópico"
```

### 📚 Conceptos Nuevos Explicados
#### 1. SNS Topic (Tópico)
Imagínalo como un "canal" de radio. Es el punto central donde se publican los mensajes. Las alarmas "hablan" al tópico, y el tópico reparte el mensaje a los oyentes.
#### 2. Subscription (Suscripción)
Es el "oyente". Define quién recibe el mensaje y por qué medio (Email, SMS, HTTP).
#### 3. El Paso de Confirmación (¡Importante!)
Por seguridad, AWS no enviará correos hasta que el dueño de la cuenta acepte. Al desplegar esto, recibirás un email de AWS Notifications con un enlace que dice "Confirm Subscription". Si no haces clic, la alarma nunca te llegará.

### 🚀 Cómo Desplegarlo
Actualiza tu stack de nuevo. CloudFormation detectará que solo estás agregando el servicio de mensajería:
```shell
aws cloudformation update-stack \
  --stack-name Mi-Infra-Con-Monitoreo \
  --template-body file://14-sns-topic-subscription.yaml

```

### 📈 Cómo probar tu Notificación
1. **Confirma el email**: Busca en tu bandeja de entrada (o Spam) el correo de confirmación de AWS.
2. **Estresa el CPU**: Usa el comando de ayer (stress --cpu 1 --timeout 600).
3. **Espera 10-15 minutos**: Cuando la métrica supere el 45%, el estado de la alarma en CloudWatch pasará a In alarm y, segundos después, ¡recibirás el aviso en tu bandeja de entrada!

### 📂 Código Adjunto
Puedes encontrar el template completo con la EC2 + Alarma + Dashboard aquí:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/14-sns-topic-subscription.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="Ouh9J_Pa2AM" provider="youtube" %}

---

### 💡 Próximos pasos
- Balanceo de Carga - Configurando un Application Load Balancer (ALB)
