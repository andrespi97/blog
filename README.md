# Mi Blog con Hugo y Dev Containers

Este es un proyecto de blog estático construido con [Hugo](https://gohugo.io/) y configurado para un desarrollo consistente usando [Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers).

## Previsualización Local

Para previsualizar tu blog localmente, asegúrate de estar dentro del Dev Container (consulta `blog_post_hugo.md` para más detalles sobre cómo configurarlo).

Una vez dentro del contenedor, abre una terminal y ejecuta el siguiente comando:

```bash
hugo server -D
```

Esto iniciará un servidor web local. Abre tu navegador y navega a `http://localhost:1313` para ver tu blog en tiempo real.

## Despliegue

El despliegue de este blog está automatizado usando [GitHub Actions](https://docs.github.com/es/actions). Cada vez que se realiza un `push` a la rama `main`, el flujo de trabajo definido en `.github/workflows/deploy.yml` construirá el sitio Hugo y lo desplegará en GitHub Pages.

## Más Información

Para una guía detallada sobre cómo configurar y trabajar con este proyecto, consulta el archivo `blog_post_hugo.md`.
