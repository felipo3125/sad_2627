# Entregable 6: Autenticidad y actualización

## Parte A · ¿Está actualizado el sistema?

### A1. Versión del sistema operativo y kernel

- **Sistema operativo:** Ubuntu 24.04.3 LTS (codename: `noble`).
- **Kernel:** `7.0.0-31-generic`.

### A2. Paquetes actualizables

Tras ejecutar `apt list --upgradable`, el sistema muestra **216 paquetes con actualizaciones pendientes**.

La mayoría pertenecen a los repositorios `noble-updates` y `noble-security`, e incluyen componentes importantes como el kernel, OpenSSL y distintas bibliotecas del sistema.

### A3. Simulación de actualización

Al ejecutar `sudo apt-get -s upgrade`, el sistema muestra qué paquetes se actualizarían sin aplicar los cambios.

En mi caso, algunas actualizaciones se aplazan debido al sistema de actualizaciones por fases (*phasing*) y aparecen paquetes retenidos (*kept back*), como el kernel y `fwupd`. El resto de los paquetes se actualizarían con normalidad.

La opción `-s` permite simular la actualización sin instalar ni modificar los paquetes.

### A4. Fin de soporte (EOL)

| Sistema | Fecha de fin de soporte | ¿Recibe parches hoy? |
|---|---|---|
| Torrent-Auditor (Ubuntu 24.04 LTS) | Mayo de 2029 para el mantenimiento estándar | Sí |
| Torrent-Vulnerable (Ubuntu 8.04) | 9 de mayo de 2013 | No |

**Pregunta A: Aunque pusieran todas las contraseñas bien, ¿qué problema sigue sin tener solución?**

El problema es que Ubuntu 8.04 está fuera de soporte (EOL) y ya no recibe actualizaciones de seguridad. Aunque las contraseñas sean seguras, el sistema operativo puede contener vulnerabilidades conocidas que no se corregirán mediante los canales habituales de actualización. Por tanto, un atacante podría aprovechar esas vulnerabilidades para acceder al equipo o comprometerlo. La solución es migrar a una versión con soporte o, mientras tanto, aislar el equipo y reducir su exposición.

---

## Parte B · ¿De dónde vienen los programas instalados?

### B1. Repositorios

El archivo `/etc/apt/sources.list` no contiene actualmente la configuración principal de los repositorios. Esta se encuentra en `/etc/apt/sources.list.d/ubuntu.sources`.

El archivo contiene dos bloques principales:

- **Repositorio principal:** utiliza el espejo español `es.archive.ubuntu.com` y las suites `noble`, `noble-updates` y `noble-backports`.
- **Repositorio de seguridad:** utiliza `security.ubuntu.com` para obtener paquetes de `noble-security`.

En ambos bloques aparecen los siguientes campos:

- `Types: deb`: indica que se descargan paquetes binarios.
- `URIs`: especifica la dirección del servidor del repositorio.
- `Suites`: indica las versiones o ramas del repositorio de las que se obtienen los paquetes.
- `Components: main restricted universe multiverse`: identifica las distintas categorías de paquetes disponibles.
- `Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg`: indica el archivo de claves que APT utiliza para comprobar las firmas de los paquetes y de los metadatos del repositorio.

Estas comprobaciones permiten verificar que los paquetes proceden de una fuente cuya clave de firma es de confianza.

### B2. Claves de confianza

**¿Qué pasaría si un atacante introduce una clave falsa en `/usr/share/keyrings/`? ¿Y si reemplaza la clave oficial de Ubuntu?**

Si un atacante consiguiera introducir una clave falsa y configurar APT para confiar en ella, podría firmar paquetes maliciosos que el sistema aceptaría como procedentes de una fuente autorizada.

Si consiguiera reemplazar la clave oficial de Ubuntu por otra controlada por él, también podría comprometer la verificación de los paquetes, siempre que el sistema utilizara la clave sustituida para comprobar sus firmas.

Por eso es importante proteger los archivos de claves, limitar los permisos de administración y verificar la autenticidad de las claves antes de confiar en ellas.

### B3. Información de un paquete

Al ejecutar `apt policy curl`, se obtiene la siguiente información:

- **Versión instalada:** ninguna (`Installed: none`).
- **Versión candidata:** `8.5.0-2ubuntu10.16`.
- **Repositorio de origen:** repositorio oficial de Ubuntu.
- **Prioridad:** `500`.

La tabla de versiones muestra las versiones disponibles del paquete, sus prioridades y los repositorios desde los que se pueden obtener.

La versión candidata es la que APT seleccionaría para instalar según las versiones disponibles y sus prioridades. En este caso, `curl` no está instalado, pero sí hay una versión candidata disponible en los repositorios configurados.

### B4. Paquetes instalados a mano

Al ejecutar `apt list --installed 2>/dev/null | grep -i local`, destaca el paquete `nessus`, que aparece como `installed,local`.

Esto indica que el paquete está instalado localmente y no se corresponde con una versión disponible en los repositorios configurados. Es posible que se instalara manualmente mediante un archivo descargado del fabricante y el comando `dpkg -i`.

En ese caso, APT no habría comprobado la firma del paquete durante esa instalación. Por ello, es importante verificar su origen y autenticidad antes de instalarlo.

### B5. Integridad de archivos

Al ejecutar `sudo dpkg -V`, aparecen avisos en archivos de `/opt/nessus/` y en `/usr/lib/systemd/system/cloud-init.service`.

El código `??5??????` indica que el resumen criptográfico MD5 del archivo no coincide con el valor registrado en la base de datos de `dpkg`. Esto señala una diferencia que conviene investigar, aunque no demuestra por sí solo que exista un ataque.

En el caso de Nessus, algunos cambios podrían deberse a sus propios mecanismos de actualización. En cambio, cualquier modificación inesperada en un archivo del sistema debe revisarse para determinar su causa.

**Pregunta B: ¿Detectaría `dpkg -V` un rootkit que también hubiera modificado la base de datos de `dpkg`? ¿Por qué conviene comparar con una copia guardada fuera del sistema?**

No necesariamente. `dpkg -V` compara los archivos instalados con la información de referencia que conserva la base de datos local. Si un atacante modifica un archivo y también altera esa información para que coincida con el cambio, la comprobación podría no detectar la manipulación.

Por eso conviene disponer de una copia de referencia guardada fuera del equipo y protegida frente a modificaciones. Así se pueden comparar los archivos y sus hashes con información que el atacante no haya podido alterar.

---

## Parte C · Verificar una descarga: hash y firma

### C1. Descarga de GNU Hello

Para esta práctica se descargan el archivo comprimido de GNU Hello, su firma digital y el llavero de claves de GNU.

Los comandos utilizados son:

```bash
mkdir -p ~/verif && cd ~/verif
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
wget https://ftp.gnu.org/gnu/gnu-keyring.gpg
```

### C2. Hash SHA-256

El hash SHA-256 obtenido para `hello-2.12.1.tar.gz` es:

```text
8d99142afd92576f30b0cd7cb42a8dc6809998bc5d607d88761f512e26c7db20
```

Al comparar el hash con el de un compañero, ambos deberían coincidir si se ha descargado exactamente el mismo archivo.

El hash permite detectar cambios en el contenido del archivo, pero por sí solo no demuestra quién lo ha publicado. Para eso es necesario verificar una firma digital o disponer de una referencia auténtica.

### C3. Verificación de la firma

La verificación con GPG devuelve el resultado `Good signature`.

- **Firmante:** Reuben Thomas (`rrt@sc3d.org`).
- **Fecha de la firma:** 29 de mayo de 2022.
- **Identificador de la clave:** `2409 3F01 6FFE 8602 EF44 9BB8 4C8E F3DA 3FD3 7230`.

GPG confirma que la firma corresponde al archivo y que este no ha cambiado desde que se firmó, siempre que la clave utilizada sea auténtica.

También aparece el aviso `WARNING: This key is not certified with a trusted signature`. Esto significa que GPG no tiene establecida una confianza personal en esa clave. No implica que la firma sea incorrecta, pero es importante verificar que la clave pertenece realmente al autor esperado.

### C4. Manipulación del archivo

Se crea una copia del archivo original y se añade un carácter al final. Después se calculan los hashes y se comprueba la firma:

```bash
cp hello-2.12.1.tar.gz hello-manipulado.tar.gz
echo "x" >> hello-manipulado.tar.gz
sha256sum hello-2.12.1.tar.gz hello-manipulado.tar.gz
cp hello-2.12.1.tar.gz.sig hello-manipulado.tar.gz.sig
gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-manipulado.tar.gz.sig hello-manipulado.tar.gz
```

El hash del archivo manipulado cambia respecto al original. En la verificación de la firma, GPG devuelve `BAD signature`.

Esto demuestra que el contenido del archivo ya no coincide con el que se firmó originalmente. Una firma digital permite detectar esta manipulación, siempre que se verifique con la clave pública auténtica del firmante.

### C5. ¿Cuándo basta con el hash y cuándo hace falta la firma?

| Situación | ¿Basta con el hash? | ¿Hace falta la firma? ¿Por qué? |
|---|---|---|
| El archivo se corrompió durante la descarga | Sí, si se dispone de un hash de referencia auténtico. | No es imprescindible para detectar la corrupción, aunque la firma también permite comprobar la integridad y la autenticidad. |
| Un atacante cambió el archivo y el hash publicado en la misma web | No, porque el atacante puede modificar ambos. | Sí. La firma permite comprobar que el archivo fue firmado con la clave privada correspondiente a la clave pública auténtica del autor. |
| Se descarga desde un espejo (*mirror*) que no es de confianza | No, si el hash procede del mismo espejo comprometido. | Sí, siempre que se disponga de la firma y se verifique con una clave auténtica obtenida de una fuente fiable. |

---

## Parte D · Política de actualizaciones para Torrent

1. **Responsable:** uno de los socios será designado para revisar las actualizaciones semanalmente y mantener un registro.
2. **Frecuencia y pruebas:** los parches de seguridad críticos se aplicarán lo antes posible, idealmente en un plazo de 24-48 horas. Las actualizaciones del ERP y del servidor de impresión se probarán antes de instalarlas en producción.
3. **Origen del software:** solo se instalarán programas procedentes de repositorios oficiales o de fabricantes cuya autenticidad se haya verificado. Las instalaciones manuales necesitarán autorización.
4. **Equipos EOL:** los sistemas que ya no reciban soporte se migrarán a versiones compatibles o se aislarán de la red hasta que puedan sustituirse.
5. **Registro y seguimiento:** se documentarán las actualizaciones realizadas, las fechas, los responsables y las incidencias detectadas.

### Debate: actualizaciones automáticas frente a manuales

Las actualizaciones automáticas permiten instalar los parches con mayor rapidez y reducen el tiempo durante el cual el sistema permanece expuesto a vulnerabilidades conocidas. Sin embargo, una actualización podría provocar incompatibilidades o interrumpir servicios importantes, como el ERP.

Las actualizaciones manuales permiten probar los cambios y controlar cuándo se instalan, lo que reduce el riesgo de interrupciones inesperadas. No obstante, requieren que una persona se encargue de ellas de forma regular; de lo contrario, los equipos pueden quedar desactualizados y expuestos a ataques.

Por tanto, Torrent debería automatizar las actualizaciones de seguridad cuando sea apropiado y probar previamente los cambios que puedan afectar a los servicios críticos.
