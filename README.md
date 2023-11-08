# Laboratorio Docker Compose

Este laboratorio se basa en el mismo ejercicio que corrimos en Replit con Python y Flask.

## Preparación

Haz un fork de este repo.

Clona el fork en tu pc.

## Paso 1

Ejecuta `docker init`.

Selecciona Python como el `application platform`.

Cuando aparezca la pregunta `What is the command to run your app?`, escribe `python main.py`.

Abre el archivo `compose.yaml`, en la línea 16, debajo de la sección `ports` agrega estas líneas:

```
    environment:
      - PORT
      - HOST
      - DB
      - DB_USERNAME
      - DB_PASSWORD
```

Asegurate que `ports` y `enviroment` estén alineados en la misma columna.

Revisa el archivo `.env`, configura las variables con los valores que usaste en el ejercicio en Replit (es decir, usa las credenciales de ElephantSQL).

Levanta la aplicación con el siguiente comando:

```
docker compose up --build
```

Revisa que todo está bien navegando a la dirección `localhost:8000`.

### Preguntas

¿Qué pasa si cambias el valor de la variable PORT?
¿Qué cambios debes hacer para cambier el port a 8080?


## Paso 2

Deten `docker compose` presionando `control-c`.

Elimina los comentarios a partir de la línea 28.

Asegurate de crear un archivo en la carpeta `db` llamado `password.txt`, coloca en este una clave cualquiera.

Modifica el archivo `.env` para que se pueda conectar al servicio `db`.

Tips: 
  - El valor para `HOST` es `db`
  - los valores que necesitan están declarados en la definición del servicio, debes hacer el "match" de las variables allí definidas con las que necesitas.

Esta vez ejecuta el comando `docker compose` de este modo:

```
docker compose up -d --build 
```

La opción `-d` permite dejar ejecutando los servicios y libera la consola.

### Actividades

Ejecuta `docker ps`. 

¿Qué obtienes?.

Ejecuta `docker images`. 

¿Cuál es el tamaño de la imagen del servidor flask?

¿Cuál es el tamaño de la imagen postgres?

¿Cuándo fueron creadas cada una de las imágenes?

Ahora ejecuta `docker compose logs -f`, esto te permite revisar el log de los contenedores.

Si navegas hacia la aplicación (http://localhost:8000/) se produce un error, revisa el log.

¿Cuál es la causa del error?

Cierra el log presionando `control-c`, luego detén la aplicación con este comando:

```
docker compose down
```

## Parte 3

Vamos a agregar un servicio adicional para migrar nuestra base de datos, esto nos permitirá crear las tablas que faltan.

Revisa el contenido de la carpeta `db/sql_migrations`, los archivos con extensión `.sql` contienen las instrucciones para crear la tabla que falta y agregarle datos.

Agrega este servicio después del servicio `db` y antes de la definición de los volumenes:

```
  flyway:
    image: flyway/flyway:7.7.0
    environment:
      - DB_USERNAME
      - DB_PASSWORD
      - DB
      - POSTGRES_PORT=5432
      - HOST
      - FLYWAY_USER=${DB_USERNAME}
      - FLYWAY_PASSWORD=${DB_PASSWORD}
      - FLYWAY_URL=jdbc:postgresql://db:5432/${DB}
      - FLYWAY_GROUP=true
    command: -locations=filesystem:/flyway/sql -connectRetries=60 migrate
    volumes:
      - ./db/sql_migrations:/flyway/sql
    depends_on:
      - db
```

Levanta la aplicación:

```
docker compose up -d --build
```

Revisa los logs para asegurarte que todo funciona correctamente.

Si todo está bien deberías navegar a la aplicación y ver los dos libros que ingresamos en la migración.

Agrega todos los cambios y has un git commit y luego un push para uw queden en tu fork.
