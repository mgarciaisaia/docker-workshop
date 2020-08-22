# Bases de Docker

## Contenedores _(containers)_

La parte más fundamental de `Docker` son los *contenedores* ("containers"). Hay mucho para decir sobre ellos, pero por ahora corramos uno:

```
docker run hello-world
```

Fácil, ¿eh? Peguémosle otra mirada a ese contenedor

```
docker ps
```

🤔 No está... Agreguemos el flag `-a`:

```
docker ps -a
```

😀 ¡Ahí está! También podemos mirarlo más de cerca con:

```
docker inspect <id-del-contenedor>
```

Ahora pongámnos serios. Corramos una versión completa de Ubuntu Linux:

```
docker run ubuntu:18.04
```

(fijate que le especificamos una versión; ya hablaremos de eso)

🤔 parece que descargó algo, pero no sé qué...

```
docker ps -a
```

¿Así que se frena después de ejecutar? Probemos otra cosa:

```
docker run -it ubuntu:18.04
```

Bien, ¡estamos dentro del contenedor! `-it` especifica que querés entrar en modo interactivo (en realidad, `i` es interactivo, y `t` es para que docker reserve una interfaz pseudo TTY para la interacción).

Después de jugar un rato, ejecutá `exit`. Miremoslo de nuevo:

```
docker ps -a
```

Así que los contenedores trabajan teniendo un único proceso principal que tiene asignado el PID 1, que ejecuta al iniciar el contenedores, y que tan pronto como ese proceso finaliza, el contenedor se frena, incluso si había otros procesos corriendo dentro.

También podrás haber notado que la primera vez que corriste `docker run ubuntu:18.04` tardó un rato, pero la segunda vez fue inmediata. Lo que ocurre es que docker intentó ejecutar un contenedor basado en la imagen `ubuntu:18.04`, y, como no la tenía localmente, la descargó desde el repositorio público. Lo que nos lleva a hablar de...

## Imágenes _(images)_

Las _imágenes_ ("images") son las plantillas que docker usa para crear los contenedores. Si te resulta familiar la [programación orientada a objetos](https://es.wikipedia.org/wiki/Programaci%C3%B3n_orientada_a_objetos), podés más o menos pensar a las imágenes como _clases_, y a los contenedores como _instancias_.

Fijate qué imágenes tenés en tu repositorio local ejecutando esto:

```
docker images
```

Consigamos otra imagen:

```
docker pull mongo
```

Este último comando _bajó_ ("pull") una imagen llamada `mongo` del [repositorio público de docker](https://hub.docker.com) a tu repositorio local. Esto es muy similar a lo que conseguís cuando hacés `git pull` desde un repositorio `git` público.

¡Bien! Así que ahora tenemos una imagen de la que no creamos ningún contenedor.

Esto es un buen resumen de las bases. Vayamos a [la próxima sección](https://github.com/mgarciaisaia/docker-workshop/tree/master/1-corriendo-contenedores).
