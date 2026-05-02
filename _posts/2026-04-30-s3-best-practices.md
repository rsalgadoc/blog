---
title: "Día 22: Mejores prácticas y optimización de costos en S3"
date: 2026-04-30
tags:
  - S3
  - BestPractices
  - FinOps
  - CloudExpert
toc: true
---

# ✨ Día 22: El toque final. Mejores prácticas y ahorro extremo
¡Llegamos al final de nuestra serie de S3! Hemos pasado de crear un simple bucket a configurar sitios web y cifrado de grado militar con KMS. Pero para ser un verdadero experto en AWS, no solo basta con saber "cómo funciona", sino con saber **cómo hacerlo de la forma más eficiente y barata posible**.

### ¿Por qué son vitales las mejores prácticas?

*   **Evitar sorpresas en la factura**: S3 es barato, pero un mal patrón de acceso o millones de archivos pequeños pueden salir caros.
*   **Rendimiento**: Saber cómo nombrar tus archivos puede mejorar la velocidad de lectura.
*   **Higiene de la nube**: Mantener buckets limpios y bien etiquetados facilita la gestión a largo plazo.

### 🛠️ El Código (CloudFormation)
Para este cierre, vamos a crear un "Bucket Maestro" que reúne lo mejor de lo que hemos visto: bloqueos de seguridad, etiquetas de costos y una regla de ciclo de vida para limpiar archivos temporales automáticamente.

#### 1. El Bucket Optimizado

```yaml
BucketProfesional:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "s3-optimizacion-final-${AWS::AccountId}"
    # 1. Seguridad Total
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
    # 2. Control de Costos (Limpieza automática)
    LifecycleConfiguration:
      Rules:
        - Id: "LimpiarUploadsIncompletos"
          Status: Enabled
          AbortIncompleteMultipartUpload:
            DaysAfterInitiation: 7
    # 3. Organización (Tags para facturación)
    Tags:
      - Key: "Proyecto"
        Value: "SerieAWS7Dias"
      - Key: "CostCenter"
        Value: "Aprendizaje"
```

### 📚 Los 5 Consejos de Oro (Mejores Prácticas)

#### 1. Nombres de archivo aleatorios al inicio
Si vas a tener miles de lecturas por segundo, no nombres tus archivos por fecha (ej. `2026-01-01-archivo.jpg`). Es mejor usar un prefijo aleatorio o un hash al principio para distribuir la carga en los servidores de AWS.

#### 2. Limpia las "Cargas Incompletas"
A veces, cuando subes un archivo grande, la subida falla a la mitad. AWS guarda esos "pedazos" y te los cobra aunque no veas el archivo. Usa la regla `AbortIncompleteMultipartUpload` (como la del código de arriba) para borrarlos automáticamente tras unos días.

#### 3. ¡Cuidado con los archivos pequeños!
S3 cobra por cada 1,000 peticiones. Si tienes 1 millón de archivos de 1 KB, pagarás mucho más en peticiones que en almacenamiento. En esos casos, es mejor agruparlos en un archivo `.zip` o `.tar`.

#### 4. Usa S3 Storage Lens
Es una herramienta gratuita en la consola de S3 que te da un panel de control con sugerencias de dónde estás desperdiciando dinero y qué buckets no tienen protección.

#### 5. Etiquetas (Tags) siempre
Nunca crees un recurso sin etiquetas como `Proyecto`, `Entorno` (Dev/Prod) o `Creador`. Cuando llegue la factura a fin de mes, sabrás exactamente qué bucket es el responsable del gasto.

### 🚀 Cómo cerrar este ciclo

Desplegar este bucket final es tu graduación en S3:

```shell
aws cloudformation deploy \
  --stack-name S3-Mejores-Practicas \
  --template-file 30-s3-best-practices.yaml
```

### 📂 Código Adjunto

Puedes descargar el template completo aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/30-s3-best-practices.yaml)

---

### 🎥 Video Tutorial

Mira el proceso paso a paso en video:

{% include video.html id="5REX9NTJT1c" provider="youtube" %}

---

### 💡 ¿Qué sigue?
Hemos terminado con S3, ¡pero el viaje serverless continúa! En el próximo módulo exploraremos **Amazon DynamoDB**, la base de datos NoSQL que escala a millones de usuarios.
