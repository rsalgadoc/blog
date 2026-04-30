---
title: "Día 21: Seguridad nivel experto: Cifrado de S3 con AWS KMS"
date: 2026-04-29
tags:
  - S3
  - Seguridad
  - KMS
  - Encryption
toc: true
---

# 🔐 Día 21: Blindaje total. Cifra tus archivos con tus propias llaves (KMS)
Hasta ahora, nuestros buckets usaban el cifrado por defecto de S3 (donde AWS gestiona las llaves). Pero para datos sensibles o entornos corporativos, necesitas tener el control total. Hoy aprenderás a usar **AWS KMS (Key Management Service)** para cifrar tus archivos con una llave que tú mismo controlas.

### ¿Por qué usar KMS en lugar del cifrado estándar?

*   **Control de acceso granular**: Puedes decidir qué usuarios pueden cifrar y quiénes pueden "desencriptar" para ver el contenido.
*   **Auditoría total**: Cada vez que alguien lee un archivo, queda un registro en AWS CloudTrail indicando que se usó la llave KMS.
*   **Cumplimiento**: Muchas normativas (como las bancarias o de salud) exigen que el cliente gestione sus propias llaves de cifrado.

### 🛠️ El Código (CloudFormation)
Este template crea una **llave maestra (KMS Key)** y configura un bucket para que la use de forma obligatoria. Si alguien intenta subir un archivo sin cifrar, S3 lo cifrará automáticamente con esta llave.

#### 1. Llave KMS y Bucket Cifrado

```yaml
# 1. Creamos la llave maestra de cifrado
MiLlaveMaestra:
  Type: AWS::KMS::Key
  Properties:
    Description: "Llave maestra para el blog - Cifrado S3"
    Enabled: true
    KeyPolicy:
      Version: "2012-10-17"
      Statement:
        - Sid: "Permitir uso total al administrador"
          Effect: Allow
          Principal:
            AWS: !Sub "arn:aws:iam::${AWS::AccountId}:root"
          Action: "kms:*"
          Resource: "*"

# 2. Creamos el bucket vinculado a esa llave
BucketUltraSeguro:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "s3-cifrado-kms-${AWS::AccountId}"
    # Bloqueo de acceso público (Mejor práctica del Día 18)
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
    # Configuración de cifrado obligatorio
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: aws:kms
            KMSMasterKeyID: !Ref MiLlaveMaestra
```

### 📚 Conceptos Nuevos Explicados

#### 1. AWS KMS (Key Management Service)
Es el servicio de AWS para crear y controlar llaves de cifrado. Piensa en él como una "caja fuerte" donde viven las llaves maestras que nunca salen de los módulos de seguridad de AWS.

#### 2. SSE-KMS (Server-Side Encryption with KMS)
A diferencia de SSE-S3 (el gratuito), aquí tú pagas por la llave ($1 USD al mes aprox.) y por el uso, pero ganas el control de la llave. Si deshabilitas la llave en la consola, **nadie** podrá leer los archivos del bucket, ni siquiera el administrador.

#### 3. Key Policy
Es el documento JSON que ves dentro de la llave. Define quién es el dueño de la llave. Es vital que el usuario `root` tenga acceso, o podrías "perder las llaves" de tu propia casa.

### 🚀 Cómo Desplegarlo

Despliega tu infraestructura con cifrado avanzado:

```shell
aws cloudformation deploy \
  --stack-name S3-KMS-Seguridad \
  --template-file 29-s3-key-encryption.yaml
```

### 📈 Cómo probar el cifrado

1. Sube un archivo a este nuevo bucket.
2. Selecciona el objeto y ve a la pestaña **Properties**.
3. Busca la sección **Server-side encryption settings**.
4. Verás que el algoritmo es **AWS Key Management Service concepts (SSE-KMS)** y aparecerá el ARN de la llave que creaste.
5. **Prueba de fuego**: Si fueras a otra cuenta de AWS e intentaras leer el archivo (aunque el bucket fuera público), no podrías porque no tienes permiso para usar la llave KMS.

### 📂 Código Adjunto

Puedes descargar el template completo aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/29-s3-key-encryption.yaml)

---

### 🎥 Video Tutorial

Mira el proceso paso a paso en video:

{% include video.html id="Sci-uwVMYUo" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 22: Mejores prácticas y optimización de costos**: El gran cierre. Repasaremos todo lo aprendido para que tu arquitectura en S3 sea impecable.

