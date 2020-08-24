# Docker swarm

El modo enjambre _(Swarm)_ de Docker te permite trabajar en serio con ambientes docker de gran escala y alta disponibilidad. Básicamente te permite manejar un clúster de máquinas como si fuera un único daemon de docker, con reemplazo de fallos _(failover)_ automático, planificación de contenedores, ruteo y otro montón de bondades.

Esta última sección te guiará en la creación de un clúster simple de Swarm y en los conceptos básicos. Sí notá que entender Docker Swarm en su totalidad está _muy_ fuera del alcance de esta guía. En cualquier caso, vayamos al grano.

## Conseguir algunos nodos

Para tener un enjambre de docker, primero necesitás un clúster de máquinas, para lo que necesitás máquinas. La forma más rápida y piola de conseguirlo es usando [`play-with-docker`](http://play-with-docker.com/) para probarlo en línea. Si preferís probarlo localmente, vas a necesitar [`docker-machine`](https://docs.docker.com/machine/) y [`Virtualbox`](https://www.virtualbox.org/). Si estás corriendo `Docker for Mac` o `Docker for Windows`, probablemente ya lo tengas instalado; en Linux vas a tener que instalar `docker-machine` por separado.

La principal diferncia es cuánto te va a llevar tener el enjambre listo. Si sólo querés hacer alguna prueba rápida, probablemente quieras usar la versión online. Si querés tener un enjambre persistente o probar alguna cosa extra, andá por el camino de ejecutar localmente (ojo que puede requerir bastantes recursos).

Elegí tu propio veneno y andá por alguno de estos dos:

#### Arenero _(sandbox)_ en línea

Navegá a [`play-with-docker`](http://play-with-docker.com/) y creá tres nodos usando el botón "+ ADD NEW INSTANCE" _(Agregar nueva instancia)_.

Saltá a [la próxima sección](https://github.com/mgarciaisaia/docker-workshop/tree/master/4-docker-swarm#enjambrate).

#### Enjambre local

Asegurate de tener `docker-machine` instalado, ejecutando:

```
docker-machine --version
```

Empezá creando el nodo maestro del clúster:

```
docker-machine create -d virtualbox manager
```

Eso creó un nodo administrador. Podés entrar por `ssh` si querés:

```
docker-machine ssh manager
```

Piola, ¿eh? Ahora dale `exit` y creemos algunos nodos trabajadores _(workers)_.

```
docker-machine create -d virtualbox worker1
docker-machine create -d virtualbox worker2
```

Ahora necesitás hablarle al daemon de docker en el nodo administrador. Podrías hacerlo entrando por `ssh`, pero hagámoslo desde afuera, que se parece más a cómo querrías hacerlo en el mundo real.

La forma rápida de hacerlo es ejecutando:

```
eval $(docker-machine env manager)
```

Este comando escribe unas variables de entorno que le indican a tu cliente docker dónde y cómo encontrar al daemon docker con el que querés que se comunique. Fijate cuáles son esas variables haciendo:

```
env | grep DOCKER
```

Notá que cada comando que le pidas a docker que corra va a ejecutarse en el dameon docker del nodo administrador, por lo que si hacés `docker images` o `docker ps` no vas a ver las cosas con las que estuvimos trabajando hasta ahora, porque ya no estás hablando con tu dameon docker local.

Para volver a trabajar con tu daemon local podés abrir una nueva terminal o hacerle `unset` a esas variables de entorno.

### Enjambrate*

Arranquemos con este enjambre. Conseguite la IP del nodo administrador. En `play-with-docker` la vas a ver en el prompt de la terminal del nodo; si estás ejecutando de manera local con `docker-machine`, suele ser `192.168.99.100`.

```
docker swarm init --advertise-addr <IP del nodo administrador>
```

Esto configura al daemon docker del nodo en modo swarm, e imprime el comando `swarm join` _(unirse al enjambre)_ que vas a necesitar para que otros nodos se unan a este enjambre. Copialo al portapapeles, que lo vas a necesitar pronto.

Verificá el estado del enjambre haciendo:

```
docker info
```

Podés ver el estado del enjambre debajo de `Swarm`.

Mirá los nodos del enjambre con:

```
docker node ls
```

Ahora hagamos que ambos nodos trabajadores se unan al clúster del enjambre: corré el comando que acabás de copiar al portapapeles dentro de cada uno de los nodos trabajadores (si estás ejecutando localmente, metete a los workers con `docker-machine ssh` y después hacé `exit` para volver a tu terminal).

Fijate que ahora tu enjambre tiene 3 nodos:

```
docker node ls
```

Ya tenés un clúster de 3 nodos actuando como enjambre 😎

## Servicios

Docker swarm trabaja con el concepto de servicios, como `docker-compose`. Creemos un servicio simple que haga ping a `docker.com`.

```
docker service create --name pinger --replicas 1 alpine ping docker.com
```

Veamos algo de información del servicio haciendo:

```
docker service inspect --pretty pinger
```

Verifiquemos su estado haciendo:

```
docker service ps pinger
```

Podés ver en qué nodo está ejecutando. Ahora, escalemos el servicio poniéndole más réplicas (cada réplica es un contendor):

```
docker service scale pinger=5
```

Si ahora hacés `docker service ps pinger` vas a ver en qué nodos están ejecutando las nuevas réplicas.

¡Ya tenés un enjambre docker local completo!

Ahora matemos uno de los nodos trabajadores y veamos cómo docker re-planifica sus contenedores: en `play-with-docker` simplemente tocá el botón "Delete" _(eliminar)_ de cualquiera de los nodos trabajadores. Si estás ejecutando localmente, hacé `docker-machine rm worker2`.

Ahora hacé `docker service ps pinger` repetidamente para ver cómo van apareciendo en los otros nodos automáticamente. ¿Copado, eh?

Ya tenés una aplicación resiliente y distribuída corriendo en un clúster de docker swarm ✨

## Palabras finales

Estos son sólamente los conceptos básicos de docker: vas a aprender mucho más tratando con casos reales, así que _a hackearla se ha dicho_.

Con algo de suerte este repo te ayude a [investigar más por tu cuenta](https://docs.docker.com) y tener a docker como parte de tu caja de herramientas de desarrollo y procesos de producción.

Sentite libre de actualizar/corregir cualquier cosa que veas mejorable en este repo, y si te gustó, difundilo.

¡Gracias por leer! 🙇

---------

* El título original de la sección era `Get swarmin'`, que es un tipo de expresión en inglés para indicar _"metete en la cosa"_, para distintos valores de "la cosa". Qué se yo 🤷‍
