# Guía de laboratorio Parte 1
Guía del laboratorio impartido por sidertia en las jornadas CCN-STIC

Tiempo estimado: **10 min**

Se ejecutará un exploit desarrollado por Nick Frichette el cual aprovecha una vulnerabilidad descubierta en febrero de 2019 identificada como CVE-2019-5736.
***
## ÍNDICE 📋
1. [CVE-2019-5736](#id1)
2. [Ejecución](#id12)

## Requisitos

1. Ir al directorio Parte4
````
cd ~/contenedoresyseguridad/Seccion1DockerEscape
````
2. Backup el archivo runc
````
sudo cp /usr/sbin/runc /usr/sbin/runc.backup
````


<div id='id1'></div>

## 1 CVE-2019-5736

Si se desea más información se puede consultar una explicación detallada del funcionamiento en el siguiente repositorio:

https://github.com/Frichetten/CVE-2019-5736-PoC

El funcionamiento general del exploit es el siguiente, SIEMPRE teniendo en cuenta que un atacante ha conseguido una shell en uno de los contenedores:

1. Se ejecuta el exploit en el contenedor quedando a la escucha esperando a que el binario de la shell del propio contenedor sea ejecutado.
2. Cuando desde el host se ejecute un exec en el contenedor llamando a sh, el contenedor sobreescribe runC por el payload diseñado.

Esto se realiza mediante el acceso a runc por medio del link simbólico /proc/self/exe que apuntará a runC en el momento de ejecutar el exploit en el contenedor.

Al realizar un exec desde fuera del contenedor, el exploit sobreescribe runC por el payload diseñado

En este caso, se a ejecutar el exploit en la instalación docker por defecto versión 18.09.1.
Ya se encuentra todo preparado para escapar de un contenedor sobreescribiendo runC y ejecutando el payload como root.

<div id='id12'></div>

## 2 Ejecución

1. **En el Host**. Para simular a un atacante, se ejecutará una shell y se mapeará el exploit compilado en un contenedor de base ubuntu en la carpeta tmp. 
````
sudo chmod 777 /home/administrator/contenedoresyseguridad/Seccion1DockerEscape/main

docker run --name contcomprometido -v /home/administrator/contenedoresyseguridad/Seccion1DockerEscape/main:/tmp/main -it ubuntu /bin/bash
````
2. **En la shell del contenedor comprometido** recién creado, se debe cambiar de directorio a tmp y ejecutar el exploit.
````
cd /tmp
./main
````
3. **Abrir una nueva sesión ssh con el host.** No se debe cerrar ni salir de la shell del punto 2. A continuación, ejecutar /bin/sh en el contenedor comprometido, momento en el que se sobreescribirá runC por el payload y se ejecutará con permisos de root en el Host.
````
docker exec -it contcomprometido /bin/sh
````
4. Se puede observar que en la **shell de contcomprometido** obtiene la salida overwritten succesfully.
**En el host** se puede visualizar el fichero /tmp/ShadowOwnedBySidertia , el cual es una copia del fichero de contraseñas shadow del host.
