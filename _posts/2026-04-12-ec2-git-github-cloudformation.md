---
title: "Día 4: Desplegando una Aplicación Web desde GitHub"
date: 2026-04-12
tags: 
  - AWS
  - CloudFormation
  - EC2
  - Git
toc: true
---

# 🌐 Día 4: Desplegando tu Sitio Web desde GitHub

Hoy llevamos la automatización al siguiente nivel. En lugar de crear páginas HTML manualmente, **clonaremos un repositorio Git** y desplegaremos una aplicación web real. Así simulamos un flujo de trabajo profesional donde el código está versionado en GitHub.

### ¿Por qué es importante esto?
Esperar a que alguien suba archivos manualmente es lento y propenso a cambios perdidos. Usar Git nos permite:
1. Descargar el código fuente directamente desde GitHub.
2. Mantener el sitio sincronizado con cambios en el repositorio.
3. Personalizar el contenido con scripts (sed, awk, etc.).
4. Simular un pipeline CI/CD básico.
5. Escalar aplicaciones web profesionales en minutos.

---

### 🛠️ El Código (CloudFormation)
Aquí está el UserData mejorado que clona un repositorio y lo despliega:

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

  # Instalar git y clonar el repo
  sudo dnf install git -y
  git clone https://github.com/rsalgadoc/html5up-photon.git /tmp/mi-sitio

  cd /tmp/mi-sitio

  sed -i "s/<strong>Photon<\/strong>/<strong>$(hostname -f)<\/strong>/g" index.html

  # Limpiar la carpeta de nginx y mover tus archivos
  sudo rm -rf /usr/share/nginx/html/*
  sudo cp -r /tmp/mi-sitio/* /usr/share/nginx/html/

  # Reiniciar Nginx para aplicar cambios
  systemctl restart nginx
```

---

### 📚 Conceptos Nuevos Explicados

#### 1. **git install** - Instalar Git
Como cualquier otro software, necesitamos instalar Git primero:

```bash
dnf install git -y
```

Git es un controlador de versiones que permite descargar código desde repositorios.

#### 2. **git clone** - Descargar un Repositorio
Clone copia un repositorio completo desde GitHub a tu sistema local:

```bash
git clone https://github.com/usuario/repositorio.git /ruta/destino
```

En nuestro caso:
```bash
git clone https://github.com/rsalgadoc/html5up-photon.git /tmp/mi-sitio
```

Esto descarga el theme "Photon" (una plantilla HTML5 profesional) a `/tmp/mi-sitio`.

**Ventaja**: No necesitas mantener archivos HTML. El código está siempre sincronizado con GitHub.

#### 3. **cd** - Cambiar Directorio
Movemos a la carpeta del repositorio descargado:

```bash
cd /tmp/mi-sitio
```

Ahora cualquier comando se ejecutará dentro de esta carpeta.

#### 4. **sed** - Buscar y Reemplazar
`sed` (Stream Editor) nos permite modificar archivos con expresiones regulares:

```bash
sed -i "s/BUSCAR/REEMPLAZAR/g" archivo.html
```

Desglose:
- `-i`: Modifica el archivo "in-place" (en el mismo lugar)
- `s`: Sustituye (search and replace)
- `g`: Global (reemplaza todas las ocurrencias, no solo la primera)

En nuestro caso:
```bash
sed -i "s/<strong>Photon<\/strong>/<strong>$(hostname -f)<\/strong>/g" index.html
```

Esto reemplaza el texto "Photon" por el nombre de la instancia EC2. Así cada servidor muestra su propio nombre.

#### 5. **rm -rf** - Eliminar Recursivamente
Borra directorios completos con todo su contenido:

```bash
rm -rf /ruta/directorio
```

- `-r`: Recursivo (borra también subcarpetas)
- `-f`: Force (no pide confirmación)

En nuestro caso:
```bash
sudo rm -rf /usr/share/nginx/html/*
```

Limpia la carpeta por defecto de Nginx (`/usr/share/nginx/html/`).

⚠️ **Cuidado**: `rm -rf` es destructivo. Asegúrate de saber qué estás borrando.

#### 6. **cp -r** - Copiar Recursivamente
Copia directorios completos:

```bash
cp -r /origen/* /destino/
```

- `-r`: Recursivo (copia también subcarpetas)
- `/*`: El asterisco significa "todo lo dentro de esta carpeta"

En nuestro caso:
```bash
sudo cp -r /tmp/mi-sitio/* /usr/share/nginx/html/
```

Copia el sitio clonado a la carpeta de Nginx.

---

### 🚀 Cómo Desplegarlo

Guarda el código como `12-ec2-git-clone-site.yaml`.

```shell
aws cloudformation create-stack \
  --stack-name Mi-Sitio-Web \
  --template-body file://12-ec2-git-clone-site.yaml
```

Especifica los parámetros:

```shell
aws cloudformation create-stack \
  --stack-name Mi-Sitio-Web \
  --template-body file://12-ec2-git-clone-site.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=dev \
    ParameterKey=VpcId,ParameterValue=vpc-xxxxx \
    ParameterKey=SubnetId,ParameterValue=subnet-xxxxx
```

Una vez desplegado, visita en tu navegador:

```
http://<IP-ELASTICA>
```

Deberías ver un sitio web profesional (Photon theme) con el nombre de la instancia personalizado en el encabezado.

**Modelo Production-Ready**: Puedes:
- Cambiar el repositorio a tu propio código
- Agregar más comandos sed para personalizar contenido
- Ejecutar comandos de build (npm install, python setup, etc.)

---

### 📂 Código Adjunto
Puedes encontrar el template completo en mi repositorio:
[Ver archivo en GitHub]({{ site.baseurl }}/code-samples/2026/04/12-ec2-git-clone-site.yaml)

---

### 🎥 Video Tutorial
Mira el proceso paso a paso en video:

{% include video.html id="" provider="youtube" %}

---

### 💡 Próximos pasos
- Clonar tu propio repositorio en lugar del ejemplo
- Agregar más personalizaciones con sed y otros comandos
- Implementar un webhook para actualizar el sitio cuando cambies el código en GitHub
