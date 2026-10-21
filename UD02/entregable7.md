# Entregable 7: Detección y eliminación de malware

**Módulo:** SAD · 2.º ASIX  
**Práctica:** Malware en un entorno controlado  
**Caso:** Estudio Técnico Torrent S.L.  
**Criterios de evaluación:** CE d) y CE e)

## Parte A · Clasificación de amenazas

Esta parte ya se realizó en el Entregable 5. Se remite a dicho documento para consultar la clasificación de las amenazas y las respuestas correspondientes.

---

## Parte B · Preparar un entorno controlado

Antes de analizar el programa sospechoso, se preparó la máquina virtual Torrent-Auditor para reducir los riesgos durante las pruebas.

| Medida de aislamiento | ¿Hecho? | ¿Qué riesgo evita? |
|---|---|---|
| Sin carpetas compartidas ni portapapeles | Sí | Reduce el riesgo de que archivos o contenido malicioso pasen entre la máquina virtual y el equipo anfitrión. |
| Snapshot previo | Sí (`limpio-antes-malware`) | Permite restaurar el estado anterior si se producen cambios no deseados o se daña el sistema. |
| Sin red al ejecutar el programa sospechoso | Sí | Evita que el programa se comunique con servidores externos, descargue otros componentes maliciosos o ataque a otros equipos accesibles por la red. |
| Máquina sin datos personales ni credenciales | Sí | Reduce las consecuencias de una posible filtración de información o del robo de credenciales. |

El aislamiento es importante porque permite analizar el comportamiento de un programa sospechoso sin exponer innecesariamente el equipo anfitrión ni otros sistemas.

---

## Parte C · El fichero EICAR

EICAR es un archivo de prueba inofensivo que permite comprobar si un antivirus detecta una amenaza simulada. No es malware real y se utiliza para verificar el funcionamiento de las herramientas de seguridad.

Se creó manualmente el archivo `eicar.txt` dentro de la máquina virtual.

**Hash SHA-256 obtenido:**

```text
131f95c51cc819465fa1797f6ccacf9d494aaaff46fa3eac73ae63ffbdfd8267
```

Al buscar este hash en VirusTotal, numerosos motores antivirus detectaron el archivo. Esto es lo esperado, ya que EICAR está diseñado precisamente para activar las detecciones de los antivirus. La búsqueda se realizó utilizando únicamente el hash, sin subir el archivo.

---

## Parte D · Instalar y usar ClamAV

### D2. Actualización de firmas

Se instaló ClamAV y se actualizó su base de datos de firmas mediante `freshclam`. La base de datos contiene información que permite reconocer amenazas conocidas.

**Fecha y versión de la base de datos:**

```text
ClamAV 1.5.4/28160/Wed Oct 21 06:25:49 2026
```

### D3. Escaneo del fichero EICAR

Se ejecutó el comando `clamscan ~/eicar.txt`. ClamAV detectó el archivo como una amenaza de prueba. El nombre de la detección fue `Eicar-Signature` y el resumen mostró `Infected files: 1`.

**Salida obtenida:**

```text
/home/linux/eicar.txt: Eicar-Signature FOUND
----------- SCAN SUMMARY -----------
Known viruses: 3628180
Engine version: 1.5.4
Scanned directories: 0
Scanned files: 1
Infected files: 1
Data scanned: 69 B
Data read: 69 B (ratio 1.00:1)
Time: 15.871 sec (0 m 15 s)
```

Este resultado demuestra que ClamAV reconoce el archivo EICAR mediante sus firmas de detección.

### D4 y D5. Escaneo y cuarentena

Se escaneó recursivamente la carpeta personal para buscar archivos infectados. Después se creó una carpeta de cuarentena y se utilizó la opción `--move` para mover allí los archivos detectados.

Se ejecutaron los siguientes comandos:

```bash
mkdir -p ~/cuarentena
clamscan -r -i -v --move="$HOME/cuarentena" ~
ls -l ~/cuarentena
```

El archivo `eicar.txt` fue movido a `~/cuarentena`.

**Resumen del escaneo:**

```text
----------- SCAN SUMMARY -----------
Known viruses: 3628180
Engine version: 1.5.4
Scanned directories: 446
Scanned files: 3500
Infected files: 1
Data scanned: 431.31 MiB
Data read: 237.89 MiB (ratio 1.81:1)
Time: 205.531 sec (3 m 25 s)
```

**Comprobación de la cuarentena:**

```text
$ ls -l ~/cuarentena
-rw-rw-r-- 1 linux linux 69 Oct 21 20:25 eicar.txt
```

La opción `-r` permite recorrer las subcarpetas, `-i` muestra los archivos infectados y `-v` activa el modo detallado. La opción `--move` mueve los archivos detectados al directorio indicado.

### D6. Detección de rootkits con rkhunter

Se instaló y ejecutó `rkhunter` para buscar indicios de posibles rootkits y otras modificaciones sospechosas del sistema. Durante el análisis aparecieron varios avisos (*warnings*).

La valoración inicial es que algunos podrían corresponder a comportamientos legítimos del sistema:

- **`/usr/bin/lwp-request`:** es un programa asociado a Perl. Su detección como script no demuestra por sí sola que exista malware.
- **Usuario y grupo `postfix`:** pueden ser legítimos si el servicio de correo correspondiente está instalado.
- **Archivos ocultos en `/etc`:** nombres como `.updated` o `.resolv.conf...` deben comprobarse antes de considerarlos sospechosos.

Estos avisos podrían ser falsos positivos, pero no es posible asegurar que todos lo sean sin revisar la salida completa y el contexto de cada archivo.

**Salida de rkhunter:**

```text
Warning: The command '/usr/bin/lwp-request' has been replaced by a script: /usr/bin/lwp-request: Perl script text executable
Warning: User 'postfix' has been added to the passwd file.
Warning: Group 'postfix' has been added to the group file.
Warning: Changes found in the group file for group 'postdrop':
Warning: Hidden file found: /etc/.resolv.conf.systemd-resolved.bak: ASCII text
Warning: Hidden file found: /etc/.updated: ASCII text
```

En conclusión, `rkhunter` sirve como herramienta complementaria para buscar indicios de compromiso, pero sus avisos requieren interpretación. Un resultado con advertencias no demuestra por sí solo que exista un rootkit.

### Pregunta D · ¿Por qué es mejor mover a cuarentena que borrar directamente?

Mover un archivo a cuarentena permite aislarlo sin destruirlo inmediatamente. Si el antivirus comete un falso positivo y marca por error un archivo importante del ERP de Torrent, borrarlo podría interrumpir el servicio o provocar una pérdida de información.

En cambio, la cuarentena permite que un administrador revise el archivo y determine si realmente es malicioso. Si se confirma que la detección fue un error, podrá recuperarlo de forma controlada.

---

## Parte E · Análisis estático de `sospechoso.sh`

El análisis estático consiste en examinar un archivo sin ejecutarlo para intentar comprender su comportamiento. Según el análisis del script, se identificaron las siguientes acciones:

| Acción identificada | Explicación |
|---|---|
| Crear una carpeta oculta | Crea `/tmp/.cache_sistema`, donde guarda elementos relacionados con su ejecución. |
| Escribir un registro | Genera un archivo de log para registrar información de su actividad. |
| Copiarse a sí mismo | Crea una copia llamada `actualizador.sh`, que después utiliza para ejecutarse. |
| Iniciar un servidor web | Ejecuta un servidor HTTP de Python en el puerto 8081, accesible desde localhost según lo observado. |
| Modificar el crontab | Añade una tarea `@reboot` para intentar ejecutarse automáticamente al iniciar el sistema. |

Estas acciones pueden ser indicios de comportamiento sospechoso, especialmente la creación de mecanismos de persistencia y la ejecución automática tras un reinicio. Sin embargo, el comportamiento debe valorarse en su contexto.

**¿Crea ficheros?**

Sí. Crea una carpeta oculta, un archivo de registro y una copia del propio script.

**¿Abre puertos?**

Sí. Inicia un servidor HTTP en el puerto 8081. En la prueba, el servicio estaba destinado a localhost, por lo que no debe interpretarse automáticamente como accesible desde otros equipos.

**¿Modifica algo para persistir?**

Sí. Añade una tarea al `crontab` mediante `@reboot`, con el objetivo de volver a ejecutar el script al iniciar el sistema.

**Si fuera un archivo `.exe`, ¿qué habría que hacer?**

Un ejecutable `.exe` suele ser un archivo binario, por lo que no se puede interpretar directamente con `cat` como si fuera un script de texto.

Se pueden utilizar herramientas como `file` para identificar su tipo, `strings` para extraer cadenas de texto legibles y Ghidra o IDA para realizar un análisis más profundo mediante desensamblado o descompilación. El análisis debe hacerse sin ejecutar el archivo sospechoso.

---

## Parte F · Análisis dinámico

El análisis dinámico consiste en observar el comportamiento de un programa durante su ejecución. Para ello, se comparó el estado de la máquina virtual antes y después de ejecutar `sospechoso.sh`.

### Informe de comportamiento

| ¿Qué hace? | Evidencia | ¿Qué fase del ataque recuerda? |
|---|---|---|
| Crea ficheros o carpetas | El `diff` entre `antes_tmp.txt` y `despues_tmp.txt` muestra la nueva carpeta `.cache_sistema`. | Instalación o ejecución. |
| Abre un puerto o proceso | El `diff` de procesos muestra `python3 -m http.server 8081` y el de puertos muestra el puerto 8081 en escucha. | Posible puerta trasera o servicio de comunicación. Podría relacionarse con mando y control (C2), aunque la escucha local por sí sola no demuestra que exista comunicación con un atacante. |
| Intenta persistir | El `diff` entre `antes_cron.txt` y `despues_cron.txt` muestra la entrada `@reboot /bin/bash /tmp/.cache_sistema/actualizador.sh`. | Persistencia. |

### Erradicación

Para eliminar los elementos creados durante la prueba, se detuvo el proceso de Python, se eliminó la tarea programada y se borró la carpeta creada por el script.

Se utilizaron los siguientes comandos como referencia para las comprobaciones:

```bash
ss -tulpn
crontab -l
ls -la /tmp/.cache_sistema
```

El proceso de Python se detuvo mediante `kill 169768`, utilizando el PID identificado durante la práctica.

Se eliminó la entrada de persistencia creada por el script y se borró la carpeta `/tmp/.cache_sistema`. Finalmente, se restauró el snapshot para devolver la máquina virtual al estado anterior a las pruebas.

**Nota:** el PID es específico de la ejecución. Para repetir el procedimiento, primero debe identificarse el PID actual. También es preferible eliminar únicamente la entrada del laboratorio del `crontab`, en lugar de borrar todas las tareas programadas del usuario.

### Pregunta F · ¿Cuál es el equivalente en un servidor real si nadie hizo la foto inicial? ¿Qué relación tiene con el inventario de servicios?

Si no existe una captura inicial o línea base, resulta más difícil identificar los cambios comparando el estado anterior y el posterior. En un servidor real se utilizan inventarios de procesos, servicios, puertos y tareas programadas, junto con registros y herramientas de monitorización.

El inventario de servicios permite conocer qué procesos y puertos son habituales y cuáles aparecen de forma inesperada. También se pueden emplear herramientas como SIEM e IDS para analizar eventos y detectar comportamientos sospechosos.

Por tanto, el inventario funciona como referencia para identificar desviaciones respecto al funcionamiento normal del sistema. No sustituye por completo a una línea base histórica, pero ayuda a reconocer elementos desconocidos o no autorizados.

---

## Parte G · Preventivo o paliativo

| Medida | Clasificación | Justificación |
|---|---|---|
| Mantener el sistema actualizado | Preventiva | Corrige vulnerabilidades conocidas antes de que puedan ser explotadas. |
| ClamAV escaneando a demanda un disco | Paliativa | Permite detectar archivos sospechosos que ya están presentes en el sistema. |
| Descargar software solo de repositorios oficiales | Preventiva | Reduce el riesgo de instalar software manipulado o de procedencia desconocida. |
| Copia de seguridad semanal desconectada de la red | Paliativa | Permite recuperar datos después de una infección, un borrado o un ataque. |
| Cuenta de usuario sin privilegios de administrador | Preventiva | Limita las acciones que puede realizar un programa malicioso si se ejecuta con los permisos de esa cuenta. |
| rkhunter tras sospechar una intrusión | Paliativa | Ayuda a buscar indicios de compromiso una vez que existe una sospecha. |
| Formación del personal contra el phishing | Preventiva | Reduce la probabilidad de que los usuarios caigan en engaños que provoquen una infección o el robo de credenciales. |

La diferencia principal es que las medidas preventivas intentan reducir la probabilidad de que ocurra un incidente, mientras que las paliativas ayudan a detectar, contener o reducir sus consecuencias. Algunas medidas pueden cumplir ambas funciones según cómo se utilicen.

---

## Conclusión

La práctica ha permitido comprobar cómo se pueden detectar amenazas mediante un antivirus, analizar indicios de rootkits y observar el comportamiento de un script sospechoso en un entorno controlado.

EICAR permite comprobar la detección de una amenaza de prueba sin utilizar malware real. ClamAV identificó el archivo y permitió aislarlo mediante cuarentena. Por otro lado, el análisis estático y dinámico de `sospechoso.sh` permitió identificar la creación de archivos, la apertura de un puerto y un mecanismo de persistencia mediante `crontab`.

Finalmente, la práctica demuestra la importancia de combinar herramientas de detección, aislamiento, monitorización y copias de seguridad para reducir los riesgos y facilitar la recuperación ante un incidente.
