## Practica ACL

# 1. Introducción y Entorno
Para la realización de esta práctica se ha habilitado el soporte de ACL en el sistema de archivos (ext4) modificando el archivo /etc/fstab y remontando la partición raíz. Se han creado los grupos y usuarios indicados en el enunciado y se ha configurado el árbol de directorios.

# 2. Configuración de ACL - Torrent S.L.
A continuación se muestra el script (script_torrent.sh) con los comandos ejecutados para dar solución al diagrama de permisos:

```bash
#!/bin/bash

# 1. Creacion de grupos
sudo groupadd operarios
sudo groupadd ingenieria
sudo groupadd supervisores

# 2. Creacion de usuarios y asignacion a sus grupos
sudo useradd -m -g operarios alberto
sudo useradd -m -g operarios oscar
sudo useradd -m -g operarios hugo

sudo useradd -m -g ingenieria nestor
sudo useradd -m -g ingenieria victor
sudo useradd -m -g ingenieria lucia

sudo useradd -m -g supervisores carlos
sudo useradd -m -g supervisores mary
sudo useradd -m -g supervisores john

# 3. Creación del entorno (directorio base y recursos)
sudo mkdir -p /home/torrent
cd /home/torrent

# Crear script hmi_planta y aplicar permisos clásicos restrictivos 
sudo touch hmi_planta
sudo chmod 700 hmi_planta

# Crear carpeta planos con sus archivos y aplicar permisos restrictivos 
sudo mkdir planos
sudo touch planos/autoclave.dwg planos/caldera.dwg planos/destilacion.dwg
sudo chmod 700 planos

# 4. Configuracion de ACLs sobre hmi_planta
# Operarios: solo ejecutar
sudo setfacl -m g:operarios:x /home/torrent/hmi_planta
# Ingeniería: leer y ejecutar
sudo setfacl -m g:ingenieria:r-x /home/torrent/hmi_planta

# 5. Configuracion de ACLs sobre planos/ (recursivo -R)
# Operarios: leer
sudo setfacl -R -m g:operarios:r /home/torrent/planos
# Ingeniería: leer, escribir y ejecutar
sudo setfacl -R -m g:ingenieria:rwx /home/torrent/planos
# Supervisores: leer y ejecutar
sudo setfacl -R -m g:supervisores:r-x /home/torrent/planos

# 6. Configuracion de ACLs por defecto (-d) para nuevos archivos en planos/
sudo setfacl -d -m g:operarios:r /home/torrent/planos
sudo setfacl -d -m g:ingenieria:rwx /home/torrent/planos
sudo setfacl -d -m g:supervisores:r-x /home/torrent/planos

echo "Configuración completada."
```


# 3. Capturas de pantalla de Verificación
<img width="955" height="915" alt="comprobar que todo correcto" src="https://github.com/user-attachments/assets/19d99b3a-f4ea-470d-bba8-f8c19ef95796" />
<img width="1920" height="928" alt="permisos para planos" src="https://github.com/user-attachments/assets/fd9c4b21-febf-416b-90a5-0ff9d576b0c5" />


# 4. Respuestas a las Preguntas de Reflexión
1. ¿Por qué no bastan los permisos clásicos de Unix (usuario / grupo / resto) para implementar el esquema del diagrama? Pon un ejemplo concreto de este caso.

R/ los permisos clasicos de unix solo da permisos a un unico propietario, unico grupo y al resto del sistema. en este caso en concreto no nos es suficiente ya que hay usuarios en grupos que tienen permisos diferentes al resto

2. ¿Qué es la máscara (mask) de una ACL y cómo afecta a los permisos efectivos de un usuario o grupo, aunque su entrada de ACL indique más permisos?

R/ la mascara es un limite de permisos maximo que se aplica a las ACL de grupos y usuarios. actua como filtro de seguridad ya que si a un grupo le damos permisos rwx pero la mascara del archivo se configura en r-x por ejemplo, el permiso que se aplica al final sera la mascara

3. ¿Qué diferencia hay entre una ACL de acceso y una ACL por defecto? ¿Cuándo se aplica cada una?

R/ la de acceso se aplica a los archivos y irectorios que ya existen en el sistema a la hora de ejecutar el comando, mientras que las que son por defecto solo se puede aplicar a directorios y se usa para que los nuevos archivos o carpetas que se creen dentro de ese directorio hereden automaticamente los permisos

4. Si mañana se incorpora un nuevo operario, ¿qué pasos tendrías que repetir para darle exactamente el mismo acceso que al resto de operarios?

R/ crear el nuevo usuario y añadirlo al grupo de operarios con el comando
```bash
sudo usermod -aG operarios nuevo_operario
```
no haria falta ejecutar acl ya que estan aplicados completamente a ese grupo y heredara los permisos

