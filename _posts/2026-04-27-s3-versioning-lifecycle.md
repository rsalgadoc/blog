---
title: "Día 19: Versionamiento y Ciclo de Vida: El 'Control+Z' de tus archivos"
date: 2026-04-27
tags:
  - S3
  - CloudFormation
  - Backup
  - DevOps
toc: true
---

# 🚀 Día 19: ¿Borraste algo por error? No entres en pánico
Hasta ahora sabemos cómo guardar archivos de forma segura y barata. Pero, ¿qué pasa si un usuario (o un script mal configurado) borra un archivo vital? Hoy aprenderás a activar el **Versionamiento** para viajar en el tiempo y las **Reglas de Ciclo de Vida** para que S3 limpie la basura automáticamente por ti.

### ¿Por qué necesitas estas funciones?

*   **Versionamiento**: Mantiene múltiples variantes de un objeto en el mismo bucket. Si borras una, siempre puedes recuperar la anterior. Es una capa de protección contra errores humanos y Ransomware.
*   **Ciclo de Vida (Lifecycle Rules)**: Permite automatizar tareas como: "Borrar versiones antiguas después de 30 días" o "Mover archivos a Glacier después de un año". Ahorra espacio y dinero sin mover un dedo.

### 🛠️ El Código (CloudFormation)
Vamos a crear un Bucket que tenga el historial de versiones activado y una regla de limpieza para que las versiones antiguas no nos cuesten una fortuna.

#### 1. Bucket con Versiones y Reglas de Limpieza

```yaml
BucketConHistorial:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "bucket-versionado-${AWS::AccountId}"
    # 1. Activar el historial de versiones
    VersioningConfiguration:
      Status: Enabled
    
    # 2. Configurar el ciclo de vida
    LifecycleConfiguration:
      Rules:
        - Id: "LimpiezaDeVersionesAntiguas"
          Status: Enabled
          NoncurrentVersionExpiration:
            NoncurrentDays: 30 # Borra versiones viejas tras 30 días
        - Id: "BorrarArchivosTemporales"
          Status: Enabled
          Prefix: "temp/" # Solo aplica a la carpeta "temp"
          ExpirationInDays: 7 # Borra el archivo original tras 7 días
```

### 📚 Conceptos Nuevos Explicados

#### 1. Versioning (Status: Enabled)
Una vez activo, cuando subes un archivo con el mismo nombre, S3 no sobrescribe el anterior, sino que le asigna un **ID de versión**. El archivo más reciente es la "versión actual".

#### 2. Marcador de eliminación (Delete Marker)
Si borras un objeto en un bucket versionado, S3 no lo elimina físicamente. En su lugar, coloca un "marcador de eliminación". Para recuperar el archivo, solo tienes que borrar ese marcador.

#### 3. NoncurrentVersionExpiration
Es vital cuando usas versionamiento. Si no pones esta regla, guardarás miles de versiones viejas para siempre, lo que disparará tu factura. Con esta regla, S3 solo guarda las versiones "no actuales" por el tiempo que tú decidas (ej. 30 días).

#### 4. Expiration vs Transition
*   **Expiration**: Borra el archivo permanentemente.
*   **Transition**: Mueve el archivo a otra clase (como vimos el Día 17), por ejemplo, de Standard a Glacier.

### 🚀 Cómo Desplegarlo

Actualiza tu infraestructura con el siguiente comando:

```shell
aws cloudformation deploy \
  --stack-name S3-Versionamiento-D19 \
  --template-file 27-s3-versioning-lifecycle.yaml
```

### 📈 Cómo probar el "Botón de Deshacer"

1. Sube un archivo llamado `nota.txt` a tu nuevo bucket.
2. Borra el archivo desde la consola de S3. ¡Parecerá que ha desaparecido!
3. En la lista de objetos del bucket, activa el interruptor que dice **"Show versions"** (Mostrar versiones).
4. Verás tu archivo `nota.txt` y arriba un **Delete marker**. 
5. Borra el "Delete marker" y... ¡mágia! Tu archivo original volverá a aparecer en la lista normal.

### 📂 Código Adjunto

Puedes descargar el template con las reglas de ciclo de vida aquí: 
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/27-s3-versioning-lifecycle.yaml)

---

### 🎥 Video Tutorial

En este video te muestro cómo el versionamiento puede salvarte de un desastre y cómo configurar las reglas de limpieza para no pagar de más:

{% include video.html id="pXakYZ18HtM" provider="youtube" %}

---

### 💡 Próximos pasos
- **Día 20: Alojamiento de sitios web estáticos**: Convierte tu bucket en un servidor web para desplegar tu portafolio o una página HTML en segundos.
