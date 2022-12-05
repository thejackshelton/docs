---
title: Despliega tu proyecto de Astro con Deno
description: Cómo desplegar tu proyecto de Astro en la web utilizando Deno.
layout: ~/layouts/DeployGuideLayout.astro
i18nReady: true
---

Puedes desplegar tu proyecto de Astro con renderizado en el servidor (SSR) en [Deno Deploy](https://deno.com/deploy), un sistema distribuido que ejecuta JavaScript, TypeScript y WebAssembly en el edge, alrededor del mundo.

Esta guía incluye instrucciones para desplegar en Deno Deploy a través de Github Actions o de la CLI de Deno Deploy.

## Requisitos

Esta guía supone que ya tienes instalado [Deno](https://deno.land/).

## Configuración del proyecto

Tu proyecto de Astro puede ser desplegado en [Deno Deploy](https://deno.com/deploy) con renderizado en el servidor (SSR). Deno Deploy no admite sitios estáticos.

### Adaptador para SSR

Para activar SSR en tu proyecto de Astro y desplegarlo en Deno Deploy:

Añadir [el adaptador de Deno](/es/guides/integrations-guide/deno/) para activar SSR en tu proyecto de Astro con el siguiente comando `astro add` que se muestra debajo. Este instalará el adaptador y hará los cambios apropiados a tu archivo `astro.config.mjs` en un solo paso.


```bash
npx astro add deno
```

Si prefieres instalar el adaptador manualmente, sigue los siguientes dos pasos:

1. Añadir [el adaptador `@astrojs/deno`](https://github.com/withastro/astro/tree/main/packages/integrations/deno) a las dependencias de tu proyecto usando tu gestor de paquetes preferido. Si estás usando npm o no estás seguro, ejecuta esto en la terminal:

    ```bash
      npm install @astrojs/deno
    ```

2. Actualiza el archivo de configuración del proyecto `astro.config.mjs` con los cambios a continuación.

    ```js ins={3,6-7}
    // astro.config.mjs
    import { defineConfig } from 'astro/config';
    import deno from '@astrojs/deno';

    export default defineConfig({
      output: 'server',
      adapter: deno(),
    });
    ```

    Luego, actualiza el script `preview` en `package.json` con el siguiente cambio.

    ```json del={8} ins={9}
    // package.json
    {
      // ...
      "scripts": {
        "dev": "astro dev",
        "start": "astro dev",
        "build": "astro build",
        "preview": "astro preview"
        "preview": "deno run --allow-net --allow-read --allow-env ./dist/server/entry.mjs"
      }
    }
    ```

    Ahora puedes utilizar este comando para previsualizar tu proyecto de Astro en producción localmente con Deno.

    ```bash
    npm run preview
    ```

## Cómo desplegar

Puedes desplegar en Deno Deploy a través de Github Actions o utilizando la CLI (interfaz de línea de comandos) de Deno Deploy.

### Despliegue con Github Actions

Si tu proyecto está almacenado en GitHub, la [página web de Deno Deploy](https://dash.deno.com/) te guiará en la configuración de Github Actions para desplegar tu proyecto de Astro.

1. Sube tu código a un repositorio público o privado de Github.

2. Inicia sesión en [Deno Deploy](https://dash.deno.com/) con tu cuenta de Github y haz clic en [New Project](https://dash.deno.com).

3. Selecciona tu repositorio, la rama desde la que quieres desplegar, y selecciona el modo **Github Action**. (Tu proyecto de Astro requiere un paso de construcción, y no puede usar el modo Automático).

4. En tu proyecto de Astro, crea un nuevo archivo en `.github/workflows/deploy.yml` y pega el YAML a continuación. Esto es similar al YAML dado por Deno Deploy, con los pasos adicionales necesarios para tu proyecto de Astro.

    ```yaml
    name: Deploy
    on: [push]

    jobs:
      deploy:
        name: Deploy
        runs-on: ubuntu-latest
        permissions:
          id-token: write # Necesario para la autentificación con Deno Deploy
          contents: read # Necesario para clonar el repositorio

        steps:
          - name: Clonar el repositorio
            uses: actions/checkout@v2

          # ¿No usas npm? Cambia `npm ci` por `yarn install` o `pnpm i`.
          - name: Instalar las dependencias
            run: npm ci

          # ¿No usas npm? Cambia `npm run build` por `yarn build` o `pnpm run build`.
          - name: Construir Astro
            run: npm run build

          - name: Subir a Deno Deploy
            uses: denoland/deployctl@v1
            with:
              project: my-deno-project # TODO: sustituir por el nombre del proyecto Deno Deploy
              entrypoint: server/entry.mjs
              root: dist
    ```

5. Después de hacer commit de este archivo YAML, y de subir a Github en tu rama de despliegue configurada, ¡el despliegue comenzará automáticamente!

   Puedes seguir el progreso utilizando la pestaña "Actions" de tu repositorio de Github, o en [Deno Deploy](https://dash.deno.com).

### Despliegue con la CLI

1. Instala la [Deno Deploy CLI](https://deno.com/deploy/docs/deployctl).

    ```bash
    deno install --allow-read --allow-write --allow-env --allow-net --allow-run --no-check -r -f https://deno.land/x/deploy/deployctl.ts
    ```

2. Ejecuta el paso de construcción de tu proyecto de Astro.

    ```bash
    npm run build
    ```

3. ¡Ejecuta `deployctl` para desplegar!

   En el siguiente comando, sustituye `<ACCESS-TOKEN>` por tu [Token de Acceso Personal](https://dash.deno.com/user/access-tokens) y `<MY-DENO-PROJECT>` por el nombre de tu proyecto en Deno Deploy.

    ```bash
    DENO_DEPLOY_TOKEN=<ACCESS-TOKEN> deployctl deploy --project=<MY-DENO-PROJECT> --no-static --include=./dist ./dist/server/entry.mjs
    ```

    Puedes seguir todos tus despliegues en [Deno Deploy](https://dash.deno.com).

4. (Opcional) Para simplificar la construcción y el despliegue en un solo comando, añade un script `deploy-deno` en `package.json`.

    ```json ins={9}
    // package.json
    {
      // ...
      "scripts": {
        "dev": "astro dev",
        "start": "astro dev",
        "build": "astro build",
        "preview": "deno run --allow-net --allow-read --allow-env ./dist/server/entry.mjs",
        "deno-deploy": "npm run build && deployctl deploy --project=<MY-DENO-PROJECT> --no-static --include=./dist ./dist/server/entry.mjs"
      }
    }
    ```

    Entonces puedes usar este comando para construir y desplegar tu sitio Astro en un solo paso.

    ```bash
    DENO_DEPLOY_TOKEN=<ACCESS-TOKEN> npm run deno-deploy
    ```

📚 Lee más sobre [SSR en Astro](/es/guides/server-side-rendering/).