# Errores de instalación y configuración

Estos son los problemas que me encontré al desplegar Odoo 19 en mi servidor con Docker (Dockge) y al configurar los usuarios.

| Error o mensaje | Causa | Solución aplicada | Fuente consultada |
|---|---|---|---|
| `No such file or directory` / `No section: 'options'` al arrancar Odoo (el contenedor se cerraba con código 1) | Monté `./config:/etc/odoo` en el `compose.yaml` sin haber creado antes el fichero en el servidor. Docker montó una carpeta vacía encima del `odoo.conf` del contenedor y Odoo no encontró la sección `[options]` | Crear a mano `/opt/stacks/oodoo/config/odoo.conf` en el servidor, con la cabecera `[options]`, los datos de conexión a la base de datos (`db_host`, `db_user`, `db_password`) y la ruta de los addons | Gemini (IA) |
| Odoo no podía leer su fichero de configuración | El `odoo.conf` creado en el servidor pertenecía a `root` y el proceso de Odoo dentro del contenedor no tenía permisos para leerlo | Dar permisos de lectura al fichero con `sudo chmod 644 /opt/stacks/oodoo/config/odoo.conf` | Gemini (IA) |
| `database files are incompatible with server` (PostgreSQL no arrancaba) | Cambié la imagen de PostgreSQL 15 a PostgreSQL 16 conservando el volumen de datos anterior. Los ficheros de una versión mayor no son compatibles con la siguiente | Parar el stack y borrar el volumen antiguo con `sudo docker compose down -v`, para que PostgreSQL 16 creara un clúster limpio desde cero | Gemini (IA) |
| `MissingSectionHeaderError` al leer `odoo.conf` | Al pegar comandos largos en la terminal se colaron caracteres invisibles (`\x1b[200~`, del modo "bracketed paste") al principio del fichero, y Odoo no reconocía la cabecera `[options]` | Reescribir el fichero con un comando `printf` y `tee`, que vuelca el texto plano sin esos caracteres | Gemini (IA) |
| `Database creation error: Access Denied` al crear una base de datos desde la web | La contraseña maestra que escribía en el gestor de bases de datos no coincidía con la definida en la configuración del despliegue | Definir la directiva `admin_passwd` en `odoo.conf` y reiniciar el contenedor, para usar esa contraseña maestra en el gestor web | Gemini (IA) |
| Odoo y PostgreSQL parados tras un apagado inesperado del servidor; la web no cargaba | Los contenedores no tenían política de reinicio, así que no volvieron a arrancar solos | Arrancarlos a mano con `docker start db odoo` | Claude (IA) |
| `¡No puedes tener dos usuarios con el mismo inicio de sesión!` al guardar al usuario Sandro Sanvicente | Le puse como correo de acceso `admin@andalusianroast.es`, que ya usaba el administrador original | Cambiar el login de Sandro a `sandro@andalusianroast.es` | Claude (IA) |

## Comprobaciones para evitar errores

- Antes de levantar Odoo, comprobar que el puerto 8200 del servidor está libre.
- Crear los ficheros de configuración en el servidor antes de montarlos como volumen.
- No cambiar de versión mayor de PostgreSQL sobre un volumen ya usado.
- Comprobar los logs con `docker logs odoo --tail 20` cuando algo no arranca.

## Uso de IA

He usado IA (Gemini y Claude) para resolver estos errores concretos de instalación, y esa es la fuente que aparece en la tabla.
