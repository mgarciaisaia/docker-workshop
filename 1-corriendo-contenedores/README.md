# Contenedores

## Corriendo, soltando _(detaching)_ y conectándose _(attaching)_ a contenedores

¡Corramos una base de datos Mongo! Y con un nombre bonito.

```
docker run --name db mongo
```

🤔 Pero no me quiero quedar conectado a su salida estándar... Démosle CTRL+C para salir, y después borremos ese contenedor.

```
docker rm db
```

Ahora corramos un nuevo contenedor de Mongo, pero en segundo plano (en _background_), con el modificador `-d` (`d` de `detach`, despegarse, soltar).

```
docker run --name db -d mongo
```

¡Bien! Ahora revisemos la base de datos. Primero necesitamos hacer algo así como un `ssh` a dentro del contenedor. Pero no usamos _realmente_ `ssh`, si no que podemos usar `execute` para ejecutar un comando en el contenedor en modo interactivo. Algo así:

```
docker exec -it db mongo
```

Podés interpretar este comando como "Hey, `docker`, ejecutame (`exec`) el comando `mongo` dentro del contenedor llamado `db`, y hacelo de modo interactivo (`-it`) para poder conectarle mi entrada y salida estándar (`STDIN` y `STDOUT`)".

Ahora simplemente estás corriendo el comando `mongo` en el contenedor `db`. Jugá un rato, y después dale CTRL+C para salir.

Si ahora hacés `docker ps`, vas a notar que el contenedor `db` sigue ejecutando. No frenó, porque el proceso principal (el proceso `mongo` de la base de datos, con `pid 1`) sigue ejecutando. El proceso que mataste al salir de la sesión fue simplemente la consola de Mongo que ejecutaste al hacer `docker exec`.

Ahora conectémonos a él desde _otro_ contenedor.

```
docker pull gvilarino/docker-testing
```

Esto te va a descargar una aplicación `node.js` muy simple que intenta conectarse a una base de datos Mongo e informa si lo logra (podés mirar el código [acá](https://github.com/gvilarino/docker-testing)). Cuando termine de descargar, simplemente hacé:

```
docker run -d --name app gvilarino/docker-testing
```

Ahora fijate si hizo algo:

```
docker logs app
```

Parece que la aplicación está escuchando en el puerto 3000. Naveguemos a `localhost:3000`

🤔 No logra conectarse a la aplicación... Miremos más de cerca:

```
docker ps -a
```

Como podés ver bajo `PORTS` (puertos), parece que la app _está_ escuchando en el puerto 3000, pero... 😮 ¡Claro! ¡Ese es el puerto _interno_ del contenedor! Lo que necesitás es _vincular_ ("bind") el puerto del contenedor a un puerto en nuestra máquina. Así que, eliminá este contenedor de la app con `docker rm -f app` y creemos uno nuevo como corresponde.

```
docker run -d --name app -p 3000:3000 gvilarino/docker-testing
```

Ahora revisá `localhost:3000` otra vez.

Bien, parece que pudimos llegar hasta ahí, pero también parece que la app no está llegando a conectarse a la DB. Si te conectás al contenedor de la app (con `docker exec`) y revisás el archivo `index.js`, vas a notar que está intentando conectarse a un equipo llamado `db`. Nuestro contenedor de la base de datos tiene exactamente ese mismo nombre. ¿Por qué no está llegando, entonces?

## Redes docker _(networks)_

Para que un contenedor sea visible a otro, ambos deben pertenecer a la misma red _(network)_. Las redes Docker son redes privadas virtuales que Docker usa para conectar a los contenedores de manera segura y privada, como si fueran equipos reales en una red física y real.

Creemos una red simple:

```
docker network create app-network
```

Ahora revisémosla:

```
docker network ls
```

Parece que todo está en orden. Conectemos nuestros contenedores a esa red.

```
docker network connect app-network db
docker network connect app-network app
```

Ahora revisá `localhost:3000` una vez más.

😎🐳

Bien, ya estamos en condiciones de [aprender a trabajar con imágenes]()
Ok, now you're ready to [learn how to work with images](https://github.com/mgarciaisaia/docker-workshop/tree/master/2-construyendo-imagenes).
