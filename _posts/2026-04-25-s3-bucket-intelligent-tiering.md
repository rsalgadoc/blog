---
title: "Día 17: Clases de almacenamiento en S3: Optimiza costos como un pro"
date: 2026-04-25
tags:
  - S3
  - CloudFormation
  - Costos
  - FinOps
toc: true
---

# ☁️ Día 17: No pagues de más. Elige la clase de almacenamiento adecuada

Ayer creamos nuestro primer Bucket, pero por defecto todo se guarda en la clase **Standard**, la más cara. Hoy vamos a configurar **Intelligent-Tiering**, la única clase que usa Machine Learning para mover tus archivos entre capas de acceso frecuente y poco frecuente automáticamente, ahorrándote dinero sin que tengas que mover un dedo.

### ¿Por qué usar Intelligent-Tiering?

*   **Ahorro sin esfuerzo**: Mueve archivos a capas baratas si no se usan en 30 días.
*   **Sin penalizaciones**: A diferencia de otras clases, no hay cargos por recuperación de datos.
*   **Rendimiento instantáneo**: Las capas automáticas tienen la misma velocidad que la clase Standard.
*   **Ideal para datos dinámicos**: Perfecto cuando no sabes qué archivos se usarán y cuáles no.

### 🛠️ El Código (CloudFormation)

Este template hace dos cosas críticas: primero, obliga a que cualquier archivo subido se convierta a **Intelligent-Tiering** de inmediato; segundo, permite que esos archivos lleguen hasta el "congelador" (Archive) para un ahorro máximo.

#### 1. Bucket con Automatización Total

```yaml
BucketAhorroAutomatico:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "s3-intelligent-tiering-${AWS::AccountId}"
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true

    # PASO 1: Mover automáticamente todo lo que se suba a Intelligent-Tiering
    LifecycleConfiguration:
      Rules:
        - Id: "MoverAIntelligentTieringInmediatamente"
          Status: Enabled
          Transitions:
            - StorageClass: INTELLIGENT_TIERING
              TransitionInDays: 0

    # PASO 2: Configurar qué hace Intelligent-Tiering con el tiempo (Archive)
    IntelligentTieringConfigurations:
      - Id: "OptimizacionDeCostosProfunda"
        Status: Enabled
        Tierings:
          - AccessTier: ARCHIVE_ACCESS
            Days: 90 # Ahorro extremo tras 90 días sin uso
          - AccessTier: DEEP_ARCHIVE_ACCESS
            Days: 180 # Costo casi cero tras 180 días sin uso
    
    Tags:
      - Key: "AutoCostOptimize"
        Value: "True"
```

### 📚 Conceptos Nuevos Explicados

#### 1. LifecycleConfiguration (Regla de Ciclo de Vida)
Es el motor que cambia las reglas del juego. Sin esto, tus archivos se quedan en la clase Standard (cara). Con `TransitionInDays: 0`, le decimos a S3: "En cuanto entre un archivo, cámbiale la clase a Intelligent-Tiering".

#### 2. Las capas "Invisibles" (Frequent e Infrequent Access)
Dentro de Intelligent-Tiering, S3 mueve los archivos entre estas dos capas de forma 100% automática basándose en si alguien los abre o no. Tú no tienes que configurar nada para esto.

#### 3. Las capas de Archivado (Archive Access)
Esto es lo que configuramos en `IntelligentTieringConfigurations`. Son capas opcionales para archivos que no vas a tocar en meses. Ojo: recuperar archivos de aquí **no es instantáneo**.

> 💡 **Tip de Pro**: S3 cobra una pequeña tarifa de monitoreo por objeto en esta clase. Si tienes millones de archivos minúsculos (menores a 128 KB), quédate en **Standard**, ya que el costo de monitoreo podría ser mayor que el ahorro.

### 🚀 Cómo Desplegarlo

Actualiza tu infraestructura con este comando:

```shell
aws cloudformation deploy \
  --stack-name S3-Intelligent-Tiering \
  --template-file 25-s3-bucket-intelligent-tiering.yaml
```

### 📈 Cómo probarlo

1. Sube un archivo a este nuevo bucket.
2. Haz clic en el archivo y revisa sus **Propiedades**.
3. En **Storage Class**, deberías ver que ahora dice **Intelligent-Tiering** en lugar de Standard. ¡La automatización está funcionando!

### 📂 Código Adjunto

Puedes descargar el template completo y funcional aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/25-s3-bucket-intelligent-tiering.yaml)

---

### 🎥 Video Tutorial

Mira el proceso paso a paso en video:

{% include video.html id="FXOoGStituo" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 18: Seguridad: Políticas de Bucket y ACLs**: Ahora que tus archivos son baratos de guardar, vamos a asegurarnos de que nadie pueda borrarlos o verlos sin permiso.
