---
title: "Día 18: Seguridad en S3: Políticas de Bucket y ACLs"
date: 2026-04-26
tags:
  - S3
  - Seguridad
  - IAM
  - CloudFormation
toc: true
---

# 🚀 Día 18: ¡Cierra los candados! Protege tus datos en Amazon S3
Ayer aprendimos a ahorrar dinero, pero de nada sirve si tus datos están expuestos. S3 es famoso por filtraciones accidentales debido a malas configuraciones. Hoy aprenderás a usar las **Bucket Policies** y el **Bloqueo de Acceso Público** para controlar exactamente quién puede entrar en tus buckets.

### ¿Cómo se controla el acceso en S3?

*   **Block Public Access (El seguro maestro)**: Es la configuración que bloquea cualquier intento de hacer el bucket público, incluso si alguien comete un error en los permisos.
*   **Bucket Policies (Recomendado)**: Políticas basadas en JSON que definen quién puede hacer qué. Son potentes y centralizadas.
*   **Ownership Controls**: Permite desactivar las antiguas ACLs para que solo las políticas de bucket manden sobre la seguridad.

### 🛠️ El Código (CloudFormation)
Vamos a crear un Bucket que implementa "defensa en profundidad": bloqueamos todo acceso público a nivel de infraestructura y añadimos una política que obliga a usar conexiones seguras (HTTPS).

#### 1. El Bucket Seguro y su Política

```yaml
MiBucketSeguro:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "bucket-seguro-${AWS::AccountId}"
    
    # 1. SEGURO MAESTRO: Bloqueo de acceso público total
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
      
    # 2. MEJOR PRÁCTICA: Desactivar ACLs para usar solo Políticas
    OwnershipControls:
      Rules:
        - ObjectOwnership: BucketOwnerEnforced

# 3. POLÍTICA DE BUCKET: Control de acceso fino
PoliticaDeAccesoS3:
  Type: AWS::S3::BucketPolicy
  Properties:
    Bucket: !Ref MiBucketSeguro
    PolicyDocument:
      Version: "2012-10-17"
      Statement:
        # Denegar cualquier conexión que no use HTTPS
        - Sid: "EnforceHTTPS"
          Effect: Deny
          Principal: "*"
          Action: "s3:*"
          Resource: 
            - !Sub "arn:aws:s3:::${MiBucketSeguro}"
            - !Sub "arn:aws:s3:::${MiBucketSeguro}/*"
          Condition:
            Bool:
              "aws:SecureTransport": "false"
```

### 📚 Conceptos Nuevos Explicados

#### 1. PublicAccessBlockConfiguration
Es la muralla principal. Aunque intentes crear una política que permita el acceso a todo internet, si estos valores están en `true`, AWS ignorará ese permiso para protegerte de filtraciones de datos.

#### 2. Bucket Policy (JSON)
Es el documento que vive en el bucket. El campo `Condition` con `aws:SecureTransport` es vital: asegura que los datos viajen siempre cifrados entre tu computadora y los servidores de AWS.

#### 3. BucketOwnerEnforced
Al activar esto, las ACLs (permisos individuales por archivo) dejan de funcionar. Esto es genial porque centraliza todo el control en un solo lugar: tu Política de Bucket.

### 🚀 Cómo Desplegarlo

Ejecuta el despliegue de tu infraestructura segura:

```shell
aws cloudformation deploy \
  --stack-name S3-Seguridad-D18 \
  --template-file 26-s3-bucket-policy.yaml
```

### 📈 Cómo probar la seguridad

1. Ve a la consola de **S3** → Entra en tu bucket → pestaña **Permissions**.
2. Verifica que el recuadro "Block all public access" esté en **On**.
3. Revisa la sección **Bucket Policy** y observa cómo AWS resalta en rojo los permisos de "Deny" para asegurar que la regla de HTTPS esté activa.

### 📂 Código Adjunto

Puedes ver el template completo con la muralla de seguridad aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/26-s3-bucket-policy.yaml)

---

### 🎥 Video Tutorial

Mira el proceso paso a paso en video:

{% include video.html id="QYQT-5JPTiM" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 19: Versionamiento y Ciclo de Vida**: Aprende a proteger tus archivos contra borrados accidentales y a automatizar su limpieza.
