# Docker compose

Si bien docker te permite construir y conectar fácilmente distintas aplicaciones con entornos y _runtimes_ diferentes, hacerlo con más de dos o tres contenedores se vuelve tedioso. `docker-compose` te permite trabajar con un sistema de contenedores complejo de manera sencilla.

Primero fijate que tengas `docker-compose` instalado (es probable) haciendo:

```
docker-compose --version
```

Si eso falla, [instalate el `docker-compose` para tu plataforma](https://docs.docker.com/compose/install/)

Para trabajar con `docker-compose` vas a necesitar un archivo `docker-compose.yml`. Fijate que el que está en el [repo `gvilarino/docker-testing`](https://github.com/gvilarino/docker-testing).

Asegurate de no estar corriendo ningún contenedor. Luego, desde el directorio ráiz del proyecto, ejecutá:

```
docker-compose up -d
```

¡Magia! ✨🐳

Lo que acaba de ocurrir es que docker creó todos los contenedores necesarios para el sistema, los conectó correctamente en una red y les vinculó todos los puertos necesarios.

```
docker-compose ps
```

Acá podés ver una variante más simple de `docker ps`, basada en los elementos que describiste en el `docker-compose.yml`. Cada uno de esos elementos se llama _servicios_ _(services)_.

Comparalo con `docker ps`. Se parecen, ¿no? La diferencia es que los nombres de los contenedores se generan como instancias del servicio, pero el servicio en sí mismo es un concepto propio de `docker-compose`.

Navegá a `localhost:3000` y vas a ver una página familiar.

## Servicios _(services)_

Un servicio es el componente fundamental de docker-compose. Es una especificación sobre cómo queres que corran los contenedores de ese servicio a través de `docker-compose`.

La propiedad `depends_on` _("depende de")_ indica qué servicios deberían iniciarse antes de iniciar el que tiene esa propiedad. Esto ayuda a indicar la precedencia de servicios.

También podés especificar valores de las variables de entorno con la propiedad `environment` _("entorno")_.

Cambiá el valor de `MONGO_URL` a `mongodb://foo/test`.

Navegá a `localhost:3000`. Vas a ver que nada cambió... ¡Porque tenés que recrear el contenedor!

```
docker-compose up -d
```

Vas a ver que el servicio `app` se re-crea y, si volvés a navegar a `localhost:3000`, vas a ver el mensaje de error.

## `docker-compose` como una herramienta de desarrollo

Digamos que querés usar esta configuración para trabajar, así que necesitás una forma de probar los cambios de tu código usando compose. Empecemos construyendo tu propia imagen *con* compose:

Borrá la propiedad `image` del servicio `app` y agregá esta otra:

```
build: .
```

Esto le dice a docker-compose dónde buscar un contexto de construcción de imágen para ese servicio. Ahora:

```
docker-compose build
```

Vas a ver unos mensajes familiares. Cuando termine, hacé:

```
docker images
```

Fijate cómo se creó una imagen nueva con un nombre generado. Podés usar esta imagen como cualquier otra y correr un contenedor haciendo `docker run`, pero por ahora hagamos:

```
docker-compose up -d
```

Los servicios se re-crearon usando la nueva imagen.

Esto nos permite aprovechar la cache de capas en cada re-construcción, y podés hacer `docker-compose build` para cada cambio que hacés en el código... un espanto 😩

Para que tu flujo de trabajo sea rápido, necesitás poder re-crear los contenedores de los servicios con contenido nuevo sin re-construir toda la imagen. Así que en lugar de re-construir la imagen, vas a re-crear el contenedor del servicio si hay cambios en los archivos fuente. Para eso, tenés que _montar_ _(mount)_ el sistema de archivos local sobre los contenedor, así:

En la sección `app:` del archivo `docker-compose.yml` agregá:

```
volumes:
  - .:/usr/src
  - /usr/src/node_modules
```

Así es como se montan los archivos fuente en el contenedor. Excluímos _específicamente_ el directorio `node_modules` porque no queremos sobreescribirlo si llegara a haber un directorio `node_modules` en la máquina host.

Ahora hacé un cambio en el `index.js` y hacé `docker-compose up -d`. Vas a ver cómo el contenedor del servicio se re-crea con los nuevos archivos fuente sin tener que re-construir toda la imagen.

Ahora pongámonos seri@s, y [liberemos al enjambre](https://github.com/mgarciaisaia/docker-workshop/tree/master/4-docker-swarm)*.

------------

* La frase original era _unleash the swarm_, donde "unleash" es algo así como _liberar todo el potencial de_, y `Docker Swarm` es la tecnología que permite manejar grupos de servidores (un "enjambre"), pero no encontré forma de traducir esta expresión sin que pierda el 80% de su significado.
