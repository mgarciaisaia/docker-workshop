# Imágenes _(images)_

## Construyendo _(build)_ imágenes

Una imagen _(image)_ docker está formada por una o más capas _(layers)_. Cada capa se construye arriba de la anterior, y son todas inmutables. Esto significa que no podés modificar una capa existente; en cambio, creás una nueva con los cambios respecto a la anterior. Esto es similar a como funcionan los diff's de `git`.

Para construir una imagen, vas a necesitar un `Dockerfile`. Probá el de la aplicación web que estuvimos usando haciendo `git clone` del [repositorio público `gvilarino/docker-testing`](https://github.com/gvilarino/docker-testing).

En el directorio raíz de ese proyecto, hacé:

```
docker build -t test-image .
```

Yyyy.... ¡Eso es todo! 🐳

Si la ejecutás e inspeccionás su contenido, vas a ver que es lo mismo que el de `gvilarino/docker-testing`, que incluye los mismos archivos del proyecto git.

Cada instrucción (`FROM`, `RUN`, etc) en el `Dockerfile` genera una nueva capa inmutable.

## Entendiendo las capas y aprovechando la cache

Notá que si volvés a correr ese mismo comando, no va a tardar nada. Esto es porque docker cachea cada capa y evita reconstruirla si el contexto de construcción y el comando de creación de la capa no cambiaron desde el último build.

Ahora cambiemos la línea `FROM node:argon` a `FROM node:6` así vemos cómo funciona con la última versión de `node.js`*, y volvamos a buildear.

Vas a ver que ¡todas las capas vuelven a construirse! Esto es porque cambiaste la capa _base_. Como todas las capas son simplemente un diff sobre la anterior, al cambiar la capa base invalidás el cache para todas las capas siguientes.

Ahora cambiemos el mensaje de texto en `index.js` y volvamos a buildear.

Acá podemos ver que la capa base se reusa, pero todo lo demás se vuelve a hacer, incluída la instalación de dependencias. En tu flujo de trabajo cotidiano, no querés reinstalar las dependencias simplemente porque cambiaste un archivo fuente - sólo querrías hacer eso si cambiaste tu `package.json`. Así que cambiemos el `Dockerfile` para aprovechar la cache de capas.

Cambiá la línea `COPY [".", "/usr/src/"]` a `COPY ["package.json", "/usr/src/"]`

Agregá un nuevo `COPY [".", "/usr/src/"]` luego de la instrucción `RUN npm install`.

Ahora volvé a buildear (va a reconstruir todo la primera vez), después volvé a cambiar el texto en `index.js` y volvé a reconstruir la imagen.

Más rápido, ¿eh? 😎

Podés publicar esta imagen usando el comando `docker push`, pero para eso vas a necesitar una cuenta en [`hub.docker.com`](https://hub.docker.com); te dejo eso como tarea para después. Por ahora, [apreandamos a usar `docker-compose`](https://github.com/mgarciaisaia/docker-workshop/tree/master/3-docker-compose).

------

* Sí, la guía original tiene sus años
