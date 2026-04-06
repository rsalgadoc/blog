# Desafiando la Nube: 365 Días de AWS

Blog de Rodrigo Salgado sobre AWS y tecnología en la nube.

---

## 📋 Tabla de Contenidos

- [Requisitos Previos](#requisitos-previos)
- [Instalación Local](#instalación-local)
- [Ejecución Local](#ejecución-local)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Configuración en GitHub](#configuración-en-github)
- [Publicación y Despliegue](#publicación-y-despliegue)

---

## 🔧 Requisitos Previos

Para ejecutar este blog localmente, necesitas tener instalado:

- **Ruby** (versión 3.0 o superior)
  - En Windows: Descarga desde [https://rubyinstaller.org/](https://rubyinstaller.org/)
  - En macOS: `brew install ruby` o usar rbenv
  - En Linux: `sudo apt-get install ruby-full`

- **Bundler** (gestor de gemas de Ruby)
  - Se instala automáticamentecon Ruby, o ejecuta: `gem install bundler`

- **Git** - Para clonar y gestionar el repositorio

### Verificar Instalación

```bash
ruby --version
bundler --version
git --version
```

---

## 📦 Instalación Local

### 1. Clonar el Repositorio

```bash
git clone https://github.com/rsalgadoc/blog.git
cd blog
```

### 2. Instalar las Dependencias

El proyecto usa `Gemfile` para gestionar las dependencias de Ruby. Ejecuta:

```bash
bundle install
```

Esto instalará:
- **jekyll** - Generador de sitios estáticos
- **github-pages** - Gemas necesarias para GitHub Pages (incluye Jekyll y plugins)
- **minimal-mistakes-jekyll** - Tema del blog
- **jekyll-remote-theme** - Para cargar temas remotos
- **webrick** - Servidor web necesario para Ruby 3.0+

---

## 🚀 Ejecución Local

### Iniciar el Servidor de Desarrollo

```bash
bundle exec jekyll serve
```

O con opciones adicionales:

```bash
bundle exec jekyll serve --livereload --drafts
```

**Parámetros útiles:**
- `--livereload`: Recarga automática del navegador cuando cambias archivos
- `--drafts`: Incluye posts no publicados en la carpeta `_drafts/`
- `--incremental`: Reconstrucción incremental (más rápido)
- `--host 0.0.0.0`: Accesible desde otras máquinas en la red

### Acceder al Blog

Una vez que el servidor está running, abre tu navegador en:

```
http://localhost:4000/blog/
```

El sitio se regenera automáticamente cuando cambias archivos.

---

## 📁 Estructura del Proyecto

```
blog/
├── _config.yml          # Configuración principal del sitio
├── _layouts/            # Plantillas HTML
├── _pages/              # Páginas estáticas (About, Tags, etc.)
├── _posts/              # Artículos del blog (nombrados: YYYY-MM-DD-titulo.md)
├── _site/               # Sitio generado (no editar, se auto-genera)
├── assets/              # Imágenes, CSS, JavaScript
├── code-samples/        # Ejemplos de código
├── Gemfile              # Dependencias de Ruby
├── Gemfile.lock         # Versiones bloqueadas de dependencias
├── index.md             # Página de inicio
└── README.md            # Este archivo
```

### Crear un Nuevo Post

Crea un archivo en `_posts/` con el formato:

```markdown
---
layout: post
title: "Título del Post"
date: 2026-04-06
categories: AWS Cloud
tags: [tag1, tag2]
excerpt: "Resumen del artículo"
---

Contenido del post aquí...
```

---

## 🌍 Configuración en GitHub

Este blog está desplegado en **GitHub Pages**. A continuación, se detallan las configuraciones necesarias:

### 1. Requisitos del Repositorio

- El repositorio debe ser **público** para que GitHub Pages sea gratis
- El repositorio puede llamarse cualquier cosa (en este caso: `blog`)

### 2. Configuración de GitHub Pages

Ve a **Settings** → **Pages** en tu repositorio:

1. **Source (Origen)**: 
   - Branch: `main` (o la rama por defecto)
   - Folder: `/ (root)`
   - ✅ GitHub construirá automáticamente el sitio desde Jekyll

2. **Custom Domain** (Opcional):
   - Si usas un dominio personalizado, configúralo aquí
   - En este caso: `rsalgadoc.github.io/blog/`

3. **HTTPS**:
   - ✅ Mantener habilitado (recomendado)
   - GitHub proporciona certificado SSL automáticamente

### 3. Archivo `_config.yml` - Configuración Crítica

Las siguientes configuraciones en `_config.yml` son esenciales para GitHub Pages:

```yaml
theme: minimal-mistakes-jekyll          # Tema del blog
url: https://rsalgadoc.github.io/blog/  # URL completa del sitio
baseurl: ""                              # Ruta relativa (vacío si está en raíz del usuario)
repository: "blog"                       # Nombre del repositorio
```

**Importante:** El `url` debe coincidir con donde esté publicado el sitio.

### 4. Gemfile - Dependencias para GitHub Pages

```ruby
source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins
gem "jekyll-remote-theme"
gem "minimal-mistakes-jekyll"
gem "webrick"
```

La gema `github-pages` incluye todas las dependencias que GitHub Pages soporta oficialmente.

### 5. `.gitignore` - Archivos a Ignorar

GitHub Page ignora automáticamente:
- `_site/` - Carpeta generada
- `.jekyll-cache/` - Caché de construcción
- `node_modules/` - Dependencias de Node (si las hay)
- `.DS_Store` - Archivos del sistema

---

## 📤 Publicación y Despliegue

### Flujo Normal de Publicación

1. **Crear un nuevo post** en `_posts/YYYY-MM-DD-titulo.md`

2. **Probar localmente**:
   ```bash
   bundle exec jekyll serve
   ```

3. **Hacer commit y push**:
   ```bash
   git add .
   git commit -m "Nuevo post: Titulo del Post"
   git push origin main
   ```

4. **GitHub hace el resto**:
   - GitHub detecta los cambios automáticamente
   - Ejecuta Jekyll para generar el sitio
   - Publica en `https://rsalgadoc.github.io/blog/`

### Verificar el Estado de Publicación

1. Ve al repositorio en GitHub
2. **Settings** → **Pages**
3. Arriba verás un mensaje verde: "Your site is published at..."
4. En **Actions** puedes ver el estado de los builds anteriores

### Cambios Comunes

| Cambio | Tiempo | Notas |
|--------|--------|-------|
| Nuevo post | 30-60s | Generalmente rápido |
| Modificar config | 1-2min | Requiere reconstrucción completa |
| Cambios en temas | 1-2min | Los cambios de tema son globales |
| Fusionar ramas | Inmediato | Si la rama ya está lista |

---

## 🐛 Solución de Problemas

### Error: "Gem not found"

```bash
bundle install
bundle update
```

### Error: "Address already in use" (puerto 4000)

El puerto 4000 ya está en uso. Usa otro:

```bash
bundle exec jekyll serve --port 5000
```

### Cambios no se reflejan

1. Detén el servidor (`Ctrl+C`)
2. Elimina la carpeta `_site/`
3. Reinicia: `bundle exec jekyll serve`

### El sitio se ve diferente en Local vs GitHub

Asegúrate que:
- `_config.yml` tiene la URL correcta
- Ejecutas `bundle exec jekyll serve` (usa las mismas gemas que GitHub)
- No estás usando plugins no soportados por GitHub Pages

---

## 📚 Recursos Útiles

- [Documentación Official de Jekyll](https://jekyllrb.com/)
- [Documentación de GitHub Pages](https://docs.github.com/en/pages)
- [Documentación del Tema Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)
- [Sintaxis Markdown](https://www.markdownguide.org/)

---

## 📝 Notas Adicionales

- Los posts deben estar en formato Markdown (`.md`)
- Usa Front Matter (el bloque YAML al inicio) para metadatos
- Los nombres de archivo de posts deben seguir el formato: `YYYY-MM-DD-titulo.md`
- Los cambios en `_config.yml` requieren reiniciar el servidor

---

**Última actualización:** Abril 2026
