---
title: "Día 3: Automatizando el Inicio de tu EC2 con UserData"
date: 2026-04-11
tags: 
  - AWS
  - CloudFormation
  - EC2
  - UserData
toc: true
---

# 🚀 Día 3: UserData - Automatización en el Inicio

Hoy damos un paso más allá. Hasta ahora hemos creado instancias EC2 que necesitan configuración manual. Hoy aprenderemos a usar **UserData** para ejecutar comandos automáticamente cuando la instancia se inicia. Desplegaremos un servidor Nginx funcional sin tocar la consola de AWS.

### ¿Por qué es importante esto?
Configurar una instancia manualmente después de crearla es tedioso y propenso a errores. UserData nos permite:
1. Ejecutar comandos bash automáticamente al iniciar la instancia.
2. Instalar software (Nginx, Node.js, Docker, etc.) sin intervención manual.
3. Crear aplicaciones listas para usar inmediatamente después de desplegar.
4. Documentar exactamente qué se instala en cada servidor.
5. Escalar a cientos de instancias con la misma configuración.

---

### 🛠️ El Código (CloudFormation)
Esta vez, en lugar de mostrar todo el template completo, nos enfocamos en lo **nuevo**: el UserData.

En la definición de la instancia EC2, agregamos:

```yaml
UserData: !Base64 |
  #!/bin/bash
  # Actualizar sistema
  dnf update -y

  # Instalar Nginx
  dnf install nginx -y

  # Iniciar y habilitar Nginx
  systemctl start nginx
  systemctl enable nginx

  # Crear una página de prueba simple
  echo "<h1>Hola desde Nginx en AWS CloudFormation!</h1>" > /usr/share/nginx/html/index.html
  echo "<p>Instancia: $(hostname -f)</p>" >> /usr/share/nginx/html/index.html

  # Reiniciar Nginx para aplicar cambios
  systemctl restart nginx
```

---

### 📚 Conceptos Nuevos Explicados

#### 1. **UserData** - Scripts de Inicialización
UserData es un script bash que se ejecuta **una sola vez** cuando la instancia se inicia por primera vez.

- Debe comenzar con `#!/bin/bash`
- Se ejecuta como usuario root
- Los comandos se ejecutan en orden
- Cualquier error detiene la ejecución

#### 2. **!Base64** - Codificación Requerida
CloudFormation requiere que UserData esté en base64. Por eso usamos `!Base64 |`:

```yaml
UserData: !Base64 |
  #!/bin/bash
  Tu código aquí
```

El `|` en YAML significa "líneas múltiples de texto".

#### 3. **dnf** - Gestor de Paquetes
En Amazon Linux 2, usamos `dnf` (o `yum`) para instalar software:

```bash
dnf update -y      # Actualiza todos los paquetes (-y = responde "sí" automáticamente)
dnf install nginx -y  # Instala Nginx
```

#### 4. **systemctl** - Control de Servicios
systemctl permite iniciar y habilitar servicios:

```bash
systemctl start nginx    # Inicia Nginx ahora
systemctl enable nginx   # Hace que Nginx se inicie automáticamente en futuros reinicios
systemctl restart nginx  # Reinicia el servicio
```

#### 5. **echo** - Crea Contenido
Podemos crear archivos directamente con `echo`:

```bash
echo "<h1>Hola</h1>" > /ruta/archivo.html   # Crea o sobrescribe
echo "<p>Más contenido</p>" >> /ruta/archivo.html  # Agrega al final
```

Aquí creamos una página HTML simple en la raíz de Nginx (`/usr/share/nginx/html/`).

#### 6. **hostname -f** - Información de la Instancia
`hostname -f` retorna el nombre de dominio completo de la instancia, útil para debugging.

---

### 🚀 Cómo Desplegarlo

Guarda el código como `ec2-with-user-data.yaml`.

```shell
aws cloudformation create-stack \
  --stack-name Mi-EC2-Con-Nginx \
  --template-body file://ec2-with-user-data.yaml
```

Especifica los parámetros:

```shell
aws cloudformation create-stack \
  --stack-name Mi-EC2-Con-Nginx \
  --template-body file://ec2-with-user-data.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=dev \
    ParameterKey=VpcId,ParameterValue=vpc-xxxxx \
    ParameterKey=SubnetId,ParameterValue=subnet-xxxxx
```

**Nota importante**: UserData tarda algunos minutos en ejecutarse. Puedes ver el progreso en:

```shell
# Conecta por SSH a la instancia y revisa los logs:
ssh -i tu-clave.pem ec2-user@<IP-ELASTICA>

# Dentro de la instancia:
tail -f /var/log/cloud-init-output.log
```

Una vez que Nginx esté corriendo, visita en tu navegador:

```
http://<IP-ELASTICA>
```

Deberías ver la página "Hola desde Nginx en AWS CloudFormation!".

---

### 📂 Código Adjunto
Puedes encontrar el template completo en mi repositorio:
[Ver archivos en GitHub]({{ site.baseurl }}/code-samples/2026-04-11/ec2-with-user-data.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="" provider="youtube" %}

---

### 💡 Próximos pasos
- Ejecutar un script más complejo (descargar aplicación desde S3, configurar variables de entorno)

