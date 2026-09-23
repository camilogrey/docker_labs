# 🧑🏽‍💻Practica 04 - PostgreSQL en Docker para Ingeniería de Datos

## Escenario

Una empresa de ingeniería de datos recibe diariamente archivos CSV procedentes de sus sistemas de ventas.

El equipo necesita crear rápidamente una base de datos temporal para:

```
ventas.csv
     ↓
PostgreSQL en Docker
     ↓
Tabla staging_ventas
     ↓
Consultas de validación
```

En lugar de instalar PostgreSQL directamente en Ubuntu Server, vamos a ejecutarlo dentro de un contenedor Docker.

Todo se realizará desde:

```
Windows
   ↓
VS Code
   ↓ SSH
Ubuntu Server
   ↓
Docker Engine
   ↓
PostgreSQL Container
```

Este laboratorio reutiliza los comandos principales aprendidos en la Clase 1 de Docker.

---

# 1. Comprobar Docker

Desde la terminal remota de **VS Code conectada a Ubuntu Server**:

```bash
docker --version
```

Comprobamos los contenedores actuales:

```bash
docker ps
```

Y las imágenes disponibles:

```bash
docker images
```
> ![Comprobar docker](docker_postgresql_img/1.%20comprobar%20docker.png)
---

# 2. Descargar PostgreSQL

Vamos a utilizar la imagen oficial:

```bash
docker pull postgres:16
```

Comprobamos que se ha descargado:

```bash
docker images
```

Deberíamos encontrar algo parecido a:

```
REPOSITORY   TAG
postgres     16
```

Aquí estamos utilizando dos comandos vistos en la clase:

```
docker pull
docker images
```

> ![Descargar la imagen de docker](docker_postgresql_img/2.%20Descargamos%20la%20imagen%20de%20postgres.png)

---

# 3. Crear el contenedor PostgreSQL

Ejecutamos:

```bash
docker run -d --name postgres-data -e POSTGRES_PASSWORD=curso123 -e POSTGRES_DB=empresa -p 5432:5432 postgres:16
```

Vamos a analizar el comando.

## `-d`

```
-d
```

Ejecuta PostgreSQL en segundo plano.

## `-name postgres-data`

```
--name postgres-data
```

Asigna el nombre:

```
postgres-data
```

al contenedor.

## `p 5432:5432`

```
-p 5432:5432
```

Relaciona:

```
Puerto 5432 Ubuntu → Puerto 5432 PostgreSQL
```

## `e`

La opción:

```
-e
```

permite definir variables de entorno.

En este caso:

```bash
-e POSTGRES_PASSWORD=curso123
```

define la contraseña del usuario administrador de PostgreSQL.

Y:

```bash
-e POSTGRES_DB=empresa
```

hace que PostgreSQL cree inicialmente una base de datos llamada:

```
empresa
```

> ![Creamos el contenedor de postgresql y se ejecute en 2do plano](docker_postgresql_img/1.%20comprobar%20docker.png)


---

# 4. Comprobar que el contenedor está funcionando

Ejecuta:

```bash
docker ps
```

Deberíamos observar algo similar a:

```
CONTAINER ID   IMAGE         PORTS                    NAMES
abc123...      postgres:16   0.0.0.0:5432->5432/tcp   postgres-data
```

Ahora tenemos:

```
Ubuntu Server
       |
       | 5432
       ↓
Docker
       |
       ↓
PostgreSQL
       |
       ↓
Base de datos empresa
```
> ![Comprobar funcionamiento del contenedor de postgresql](docker_postgresql_img/4.%20comprobacion%20del%20funcionamiento%20del%20contenedor%20postgrest.png)

---

# 5. Consultar los logs

PostgreSQL tarda unos segundos en inicializarse.

Podemos observar el proceso con:

```bash
docker logs postgres-data
```

Entre los mensajes deberíamos terminar encontrando algo parecido a:

```
database system is ready to accept connections
```

También podemos seguir los logs en tiempo real:

```bash
docker logs -f postgres-data
```

Para salir:

```
Ctrl + C
```

El contenedor continuará funcionando.

---

> ![Revisar los logs de postgresql](docker_postgresql_img/5.%20Revisar%20los%20logs%20de%20postgres.png)

> ![Revisar los logs en real time de postgresql](docker_postgresql_img/5.%20Revisar%20los%20logs%20de%20postgres%20en%20tiempo%20real.png)


# 6. Entrar en PostgreSQL

Ahora utilizamos `docker exec`.

Ejecuta:

```bash
docker exec -it postgres-data psql -U postgres -d empresa
```

Estamos haciendo lo siguiente:

```
docker exec
       ↓
contenedor postgres-data
       ↓
ejecutar programa psql
       ↓
conectarse a BD empresa
```

El prompt debería cambiar a algo parecido a:

```
empresa=#
```

Ya estamos dentro de PostgreSQL.

---

> ![Entrar a postgresql](docker_postgresql_img/6.%20Entrar%20en%20postgresql.png)


# 7. Crear una tabla de Staging

Dentro de PostgreSQL:

```sql
CREATE TABLE staging_ventas (
    id INTEGER,
    fecha DATE,
    producto VARCHAR(100),
    cantidad INTEGER,
    precio NUMERIC(10,2)
);
```

Comprobamos la tabla:

```sql
SELECT * FROM staging_ventas;
```

Todavía estará vacía.

Salimos:

```
\q
```

---

> ![Crear la tabla de staging de ventas](docker_postgresql_img/7.%20crear%20la%20tabal%20de%20staging.png)


# 8. Crear un pequeño dataset CSV

Ahora estamos nuevamente en Ubuntu Server.

Vamos a crear un archivo de datos:

```bash
echo "id,fecha,producto,cantidad,precio" > ventas.csv
```

Añadimos algunas ventas:

```bash
echo "1,2026-09-01,Portatil,2,1200.00" >> ventas.csv
```

```bash
echo "2,2026-09-01,Monitor,5,350.00" >> ventas.csv
```

```bash
echo "3,2026-09-02,Teclado,10,75.00" >> ventas.csv
```

```bash
echo "4,2026-09-02,Raton,15,35.00" >> ventas.csv
```

```bash
echo "5,2026-09-03,Portatil,1,1350.00" >> ventas.csv
```

Visualizamos:

```bash
cat ventas.csv
```

Resultado:

```
id,fecha,producto,cantidad,precio
1,2026-09-01,Portatil,2,1200.00
2,2026-09-01,Monitor,5,350.00
3,2026-09-02,Teclado,10,75.00
4,2026-09-02,Raton,15,35.00
5,2026-09-03,Portatil,1,1350.00
```

Aquí tenemos nuestro pequeño **dataset de origen**.

---

> ![Creamos un dataset csv](docker_postgresql_img/8.%20Creamos%20un%20pequeno%20database%20csv.png)


# 9. Copiar el CSV al contenedor

Utiliza:

```bash
docker cp ventas.csv postgres-data:/tmp/ventas.csv
```

El flujo es:

```
Ubuntu Server
ventas.csv
     |
     | docker cp
     ↓
Contenedor PostgreSQL
/tmp/ventas.csv
```

Podemos comprobar que llegó:

```bash
docker exec postgres-data ls /tmp
```

Debería aparecer:

```
ventas.csv
```

También podemos visualizarlo desde fuera del contenedor:

```bash
docker exec postgres-data cat /tmp/ventas.csv
```

---

> ![Copiamosla bbd csv a docker y comprobamos](docker_postgresql_img/9.%20Copiamos%20la%20bbdd%20csv%20a%20docker%20y%20comporbamos.png)


# 10. Cargar el CSV en PostgreSQL

Entramos nuevamente:

```bash
docker exec -it postgres-data psql -U postgres -d empresa
```

Ejecutamos:

```sql
COPY staging_ventas
FROM '/tmp/ventas.csv'
DELIMITER ','
CSV HEADER;
```

PostgreSQL debería indicar:

```
COPY 5
```

Eso significa que ha cargado:

```
5 filas
```

---

> ![ingresamos de nuevo a postgresql y cargamos el csv a postgresql](docker_postgresql_img/10.%20estramos%20denuevo%20a%20postgresql%20y%20cargamos%20el%20csv%20en%20postgresql.png)


# 11. Validar los datos

En Ingeniería de Datos no basta con cargar información.

Hay que comprobarla.

Ejecutamos:

```sql
SELECT * FROM staging_ventas;
```

## Contar registros

```sql
SELECT COUNT(*)
FROM staging_ventas;
```

Resultado esperado:

```
5
```

## Calcular ventas

```sql
SELECT
    producto,
    SUM(cantidad * precio) AS importe_ventas
FROM staging_ventas
GROUP BY producto
ORDER BY importe_ventas DESC;
```

Ahora ya estamos realizando una pequeña transformación analítica:

```
CSV
 ↓
Staging
 ↓
Validación
 ↓
Agregación
```

---

> ![Validamos y hacemos calculos de con la bbdd y salida](docker_postgresql_img/11%20y%2012.%20Validamos%20y%20hacemos%20unos%20calculos%20el%20la%20bbdd%20staging%20ventas%20de%20postgresql%20y%20salimos%20de%20postgresql.png)


# 12. Salir de PostgreSQL

```
\q
```

---

# 13. Inspeccionar el contenedor

Utilizamos otro comando de la Clase 1:

```bash
docker inspect postgres-data
```

Busca visualmente información relacionada con:

```
IPAddress
Ports
State
Image
Name
```

---

> ![Inspeccionamos el contenedro](docker_postgresql_img/13.%20Inspeccionamos%20el%20contenedor%20.png)


# 14. Consultar el puerto

```bash
docker port postgres-data
```

Deberíamos obtener algo parecido a:

```
5432/tcp -> 0.0.0.0:5432
```

Es decir:

```
PostgreSQL
Container :5432
      ↑
      |
Ubuntu :5432
```

---

> ![Consultamos el puerto de Ubunto a Docker](docker_postgresql_img/14.%20Consultamos%20el%20puerto%20de%20ubuntu%20a%20docker.png)


# 15. Consultar recursos utilizados

Ejecuta:

```bash
docker stats postgres-data
```

Podemos observar:

```
CPU %
MEM USAGE
MEM %
NET I/O
```

Para salir:

```
Ctrl + C
```

---

> ![Consultamos los recursos utilizados](docker_postgresql_img/15.%20consultamos%20los%20recursos%20utilizados.png)


# 16. Detener PostgreSQL

Ejecuta:

```bash
docker stop postgres-data
```

Comprobamos:

```bash
docker ps
```

Ya no aparecerá.

Pero si ejecutamos:

```bash
docker ps -a
```

seguirá existiendo:

```
postgres-data
```

con estado similar a:

```
Exited
```

> **Detener un contenedor no significa eliminarlo.**
> 

---

> ![Detenemos el contenedor y comporbamos su existencia ](docker_postgresql_img/16.%20Detenemos%20el%20contenedor%20y%20comprobamos%20que%20se%20haya%20detenido%20pero%20que%20siga%20existiendo.png)


# 17. Volver a iniciar PostgreSQL

Ejecuta:

```bash
docker start postgres-data
```

Comprobamos:

```bash
docker ps
```

PostgreSQL vuelve a estar funcionando.

---

> ![Iniciamos postgresql y comprobamos su inicio](docker_postgresql_img/17.%20Iniciamos%20postgresql%20y%20comprobamos.png)


# 18. Comprobar si los datos siguen allí

Ejecutamos:

```bash
docker exec -it postgres-data psql -U postgres -d empresa
```

Y después:

```sql
SELECT * FROM staging_ventas;
```

Los datos siguen presentes porque simplemente hemos detenido e iniciado **el mismo contenedor**.

Salimos:

```
\q
```

Esto refuerza la diferencia entre:

```
docker stop
      ↓
contenedor permanece

docker start
      ↓
volvemos a utilizarlo
```

---

> ![Verificamos que los datso copiados sigan en contenedor y salimos](docker_postgresql_img/18.%20verificamos%20que%20los%20datso%20cpiado%20sigan%20en%20el%20contenedor%20de%20postgresql%20y%20salimos%20.png)


# 19. Reiniciar PostgreSQL

Podemos hacerlo directamente:

```bash
docker restart postgres-data
```

Comprobamos:

```bash
docker ps
```

---

> ![Reiniciamos el contenedor y verificamos su existencia](docker_postgresql_img/19.%20reiniciamos%20el%20contendero%20y%20verificamos%20su%20existencia.png)


# 20. Eliminar el contenedor

Primero:

```bash
docker stop postgres-data
```

Después:

```bash
docker rm postgres-data
```

Comprobamos:

```bash
docker ps -a
```

`postgres-data` ya no existe.

Aquí aparece una lección importante para futuras clases:

> Los datos estaban almacenados dentro del contenedor. Al eliminar el contenedor, esos datos dejan de estar disponibles con él.
> 

Esto prepara el siguiente tema:

```
Docker Volumes
```

porque allí aprenderemos cómo conseguir:

```
Eliminar contenedor
       ↓

Datos sobreviven
       ↓

Crear otro contenedor
       ↓

Recuperar los mismos datos
```

---

> ![Detenemos el contenedor, lo eliminamos y verificamos su inexistencia](docker_postgresql_img/20.%20Ahora%20al%20contendore%20los%20detenemos%20y%20lo%20eliminamos%20y%20comprobamos%20su%20inexistencia.png)


# 21. La imagen PostgreSQL todavía existe

Aunque hayamos eliminado el contenedor:

```bash
docker images
```

seguiremos teniendo:

```
postgres:16
```

Esto refuerza nuevamente:

```
IMAGEN ≠ CONTENEDOR
```

La imagen es la plantilla.

El contenedor era una instancia creada a partir de ella.

---

> ![Verificar que la imagen de postgresql existe](docker_postgresql_img/21.%20verificar%20que%20la%20imagen%20aun%20existe.png)


# 22. Eliminar la imagen

Si queremos limpiar completamente:

```bash
docker rmi postgres:16
```

Comprobamos:

```bash
docker images
```

---

> ![Eliminamos la imagen y comporbamos su inexistencia](docker_postgresql_img/22.%20eliminamos%20la%20imagen%20de%20sposgresql%20y%20comprobamos.png)


# 23. Comandos de la Clase 1 utilizados

Este laboratorio utiliza casi todos los comandos principales:

| Comando | Uso dentro del laboratorio |
| --- | --- |
| `docker --version` | Comprobar instalación |
| `docker pull` | Descargar PostgreSQL |
| `docker images` | Ver la imagen |
| `docker run` | Crear PostgreSQL |
| `docker ps` | Ver PostgreSQL activo |
| `docker ps -a` | Ver activo/detenido |
| `docker logs` | Revisar inicialización |
| `docker exec` | Ejecutar SQL dentro del contenedor |
| `docker cp` | Introducir el CSV |
| `docker inspect` | Examinar configuración |
| `docker port` | Consultar publicación 5432 |
| `docker stats` | Consultar recursos |
| `docker stop` | Detener PostgreSQL |
| `docker start` | Iniciarlo otra vez |
| `docker restart` | Reiniciarlo |
| `docker rm` | Eliminar el contenedor |
| `docker rmi` | Eliminar la imagen |

---

---

# 25. Flujo completo del laboratorio

```
ventas.csv
    │
    │ docker cp
    ▼
┌──────────────────────────┐
│ Docker Container         │
│                          │
│ PostgreSQL               │
│ ┌──────────────────────┐ │
│ │ staging_ventas       │ │
│ │                      │ │
│ │ id                   │ │
│ │ fecha                │ │
│ │ producto             │ │
│ │ cantidad             │ │
│ │ precio               │ │
│ └──────────────────────┘ │
└──────────────────────────┘
             │
             │ SQL
             ▼
      Validación
             │
             ▼
      Transformación
             │
             ▼
       Datos analíticos
```

---
