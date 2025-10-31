---
title: "Cómo Crear un Blog con Hugo y Dev Containers"
date: 2024-10-31T10:00:00Z
draft: false
---

# Cómo Crear un Blog con Hugo y Dev Containers: Guía Paso a Paso

Esta guía te mostrará cómo configurar un entorno de desarrollo completo para un blog estático usando Hugo y Dev Containers. Esto te permitirá tener un entorno de desarrollo consistente y reproducible, ideal para trabajar en cualquier máquina.

## Prerrequisitos

- [Visual Studio Code](https://code.visualstudio.com/)
- La extensión [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) para VS Code.
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y en ejecución.

## Paso 1: Creación de la Estructura del Proyecto

Primero, crea una carpeta para tu proyecto y navega hacia ella en tu terminal.

```bash
mkdir mi-blog-hugo
cd mi-blog-hugo
```

Dentro de esta carpeta, crea la siguiente estructura de directorios:

```bash
mkdir -p .devcontainer .github/workflows posts
```

- `.devcontainer/`: Contendrá la configuración de nuestro entorno de desarrollo.
- `.github/workflows/`: Aquí definiremos el flujo de trabajo de GitHub Actions para el despliegue automático.
- `posts/`: Aquí es donde escribirás tus artículos en formato Markdown.

## Paso 2: Configuración del Entorno de Desarrollo (`.devcontainer`)

Nuestro entorno de desarrollo necesita Hugo y Git. Usaremos un `Dockerfile` para definirlo.

**1. Crea el `Dockerfile`:**

Crea un archivo llamado `Dockerfile` dentro de la carpeta `.devcontainer` con el siguiente contenido. Este ejemplo está configurado para una arquitectura ARM64 (como los Mac con Apple Silicon). Si usas una arquitectura x86_64 (Intel/AMD), deberás cambiar el archivo de Hugo a descargar.

```dockerfile
# .devcontainer/Dockerfile
FROM debian:bookworm-slim

# Instala Git y las dependencias necesarias
RUN apt-get update && apt-get install -y curl git

# Instala Hugo (versión para ARM64)
RUN curl -L "https://github.com/gohugoio/hugo/releases/download/v0.125.4/hugo_extended_0.125.4_linux-arm64.deb" -o "hugo.deb" && \
    apt-get install -y ./hugo.deb && \
    rm hugo.deb
```

**2. Crea `devcontainer.json`:**

Ahora, crea el archivo `devcontainer.json` en la misma carpeta `.devcontainer`. Este archivo le dice a VS Code cómo usar el `Dockerfile` y configurar el entorno.

```json
// .devcontainer/devcontainer.json
{
  "name": "Hugo-Blog-Dev-Container",
  "build": {
    "dockerfile": "Dockerfile"
  },
  // Redirige el puerto del servidor de Hugo para previsualización
  "forwardPorts": [1313],
  "customizations": {
    "vscode": {
      // Extensiones recomendadas para TOML y Markdown
      "extensions": [
        "bungcip.better-toml",
        "DavidAnson.vscode-markdownlint"
      ]
    }
  }
}
```

## Paso 3: Abre el Proyecto en el Dev Container

Abre la carpeta de tu proyecto en VS Code. Automáticamente, VS Code detectará la configuración del `.devcontainer` y te mostrará una notificación en la esquina inferior derecha.

Haz clic en **"Reopen in Container"**.

VS Code construirá la imagen de Docker (puede tardar unos minutos la primera vez) e iniciará el entorno de desarrollo. Sabrás que estás dentro del contenedor porque la esquina inferior izquierda de VS Code se pondrá de color verde y mostrará el nombre del contenedor.

## Paso 4: Inicializa Hugo y Git

Ahora que estás dentro del contenedor, abre una terminal en VS Code (`Terminal > New Terminal`).

**1. Inicializa Hugo:**

```bash
# El flag --force es necesario porque el directorio no está vacío
hugo new site . --force
```

**2. Inicializa Git:**

```bash
git init
```

## Paso 5: Añade un Tema

Hugo necesita un tema para renderizar el contenido. Usaremos el tema [LoveIt](https://github.com/dillonzq/LoveIt) como ejemplo, añadiéndolo como un submódulo de Git.

```bash
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

## Paso 6: Configura Hugo

Modifica el archivo `hugo.toml` que se creó en la raíz de tu proyecto para que se vea así:

```toml
# hugo.toml
baseURL = "https://tu-dominio.com/"
languageCode = "es-es"
title = "Mi Increíble Blog"
theme = "LoveIt"
```

> **Nota:** Reemplaza `https://tu-dominio.com/` con la URL real donde desplegarás tu blog.

## Paso 7: Crea tu Primer Post

Usa el CLI de Hugo para crear un nuevo archivo de post. Esto generará un archivo Markdown con la estructura (front matter) necesaria.

```bash
hugo new posts/mi-primer-post.md
```

Abre el archivo `posts/mi-primer-post.md` y empieza a escribir. Verás que tiene una cabecera como esta:

```markdown
---
title: "Mi Primer Post"
date: 2025-10-31T12:00:00Z
draft: true
---

¡Hola, mundo! Este es el contenido de mi primer post.
```

> **Importante:** Para que un post sea visible en el sitio final, cambia `draft: true` a `draft: false`.

## Paso 8: Previsualiza tu Blog en Local

Puedes levantar un servidor web local para ver cómo se ve tu blog en tiempo real.

```bash
hugo server -D
```

- El flag `-D` (o `--buildDrafts`) es para que también se muestren los borradores (`draft: true`).

Abre tu navegador y ve a `http://localhost:1313`. Deberías ver tu blog con el tema LoveIt y tu primer post.

## Paso 9: Despliegue con GitHub Actions

Finalmente, vamos a automatizar el despliegue a GitHub Pages.

**1. Crea el workflow:**

Crea el archivo `deploy.yml` dentro de la carpeta `.github/workflows` con el siguiente contenido:

```yaml
# .github/workflows/deploy.yml
name: Deploy Blog to GitHub Pages

on:
  push:
    branches:
      - main # O la rama que uses como principal

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true  # Asegura que los submódulos del tema se descarguen
          fetch-depth: 0    # Necesario para Hugo

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public # El directorio que Hugo genera
```

**2. Configura GitHub Pages:**

En tu repositorio de GitHub, ve a `Settings > Pages`. En la sección "Build and deployment", selecciona `GitHub Actions` como la fuente.

**¡Y listo!** Cada vez que hagas un `push` a tu rama `main`, GitHub Actions construirá tu sitio Hugo y lo desplegará automáticamente en GitHub Pages.
