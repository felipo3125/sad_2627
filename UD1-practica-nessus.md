# Informe de auditoría de vulnerabilidades

## 1. Contexto y alcance

Estudio Técnico Torrent S.L. va a firmar un contrato de mantenimiento
informático con una nueva empresa de servicios y, antes de firmarlo,
quiere conocer el estado real de seguridad de sus sistemas. Para ello
nos ha contratado como auditores externos.

El alcance de esta auditoría se limita a un sistema de laboratorio
(Metasploitable2), una máquina virtual deliberadamente vulnerable que
representa "lo que nos podríamos encontrar" en una pyme que nunca ha
hecho una revisión de seguridad. No se ha auditado ningún sistema real
en producción, ni ninguna red ajena al laboratorio.

## 2. Metodología

- **Herramienta utilizada**: Nessus Essentials Plus.
- **Tipo de escaneo**: Basic Network Scan (descubrimiento de puertos,
  detección de servicios y identificación de vulnerabilidades conocidas).
- **Objetivo analizado**: Metasploitable2, IP 192.168.56.101.
- **Entorno**: red interna aislada de VirtualBox (`red-torrent`), sin
  acceso a Internet desde la víctima. La máquina auditora (Ubuntu con
  Nessus) dispone de un adaptador NAT únicamente para la activación de
  la licencia y la descarga de plugins.
- **Duración del escaneo**: 19 minutos.
- **Fecha**: septiembre de 2026.

## 3. Resumen de resultados

Total detectado: **70 vulnerabilidades** distribuidas:

| Severidad | Nº aproximado |
|---|---|
| Crítica | 6 |
| Alta | 1 |
| Media | 7 |
| Baja | pocas |
| Informativa | 50 |

La mayor parte son informativos (banners, versiones,
configuraciones detectadas), pero el pequeño grupo de críticas y altas
concentra el riesgo real del sistema: backdoors, protocolos obsoletos y
servicios mal configurados.

## 4. Vulnerabilidades clasificadas

| Vulnerabilidad | Severidad / CVSS | Origen | Breve descripción |
|---|---|---|---|
| Bind Shell Backdoor Detection (Plugin 51988) | Crítica / 10.0 | Implementación | Hay una shell escuchando en un puerto sin pedir autenticación. El binario del servicio fue manipulado (o se introdujo código malicioso) para abrir una puerta trasera: cualquiera que se conecte ejecuta comandos con los privilegios de ese servicio. |
| VNC Server 'password' Password (Plugin 61708) | Crítica / 10.0 | Uso | El servidor VNC está configurado con la contraseña `password`, trivial de adivinar. No es un fallo del software, sino una mala configuración del administrador. |
| SSL Version 2 and 3 Protocol Detection (Plugin 20007) | Crítica / 10.0 | Diseño | El servicio acepta conexiones cifradas con SSL 2.0 y/o 3.0, protocolos obsoletos con fallos criptográficos estructurales (padding inseguro, renegociación insegura, POODLE, DROWN). El fallo no está en una versión concreta del software, sino en el diseño del propio protocolo. |

**Criterio para calificar cada una:**

- **Implementación** → el fallo está en el código concreto de esa
  versión del software (código manipulado, backdoor, bug).
- **Uso** → el software funciona bien, pero está mal configurado o se
  ha dejado con valores por defecto (contraseñas, puertos, permisos).
- **Diseño** → el fallo está en cómo está pensado el protocolo o
  servicio, aunque la implementación sea correcta.

## 5. Análisis en profundidad

### Bind Shell Backdoor Detection (Plugin ID 51988)

**Qué es**: una shell (intérprete de comandos) que está escuchando en
un puerto de la máquina víctima y que no pide usuario ni contraseña
para acceder. Es una de las señales más claras de que un sistema ha
sido comprometido o de que trae una puerta trasera de fábrica.

**Cómo se podría explotar**: un atacante solo necesita conocer la IP
de la víctima y el puerto donde escucha esa shell. Con una herramienta
tan básica como `netcat` (`nc <IP> <puerto>`) se conecta y obtiene
directamente una consola con los privilegios del usuario que ejecuta
el servicio (habitualmente root). Desde ahí puede leer y modificar
archivos, crear usuarios, instalar más malware o pivotar a otras
máquinas de la red. No hace falta exploit, credenciales, técnica
avanzada.

**Cómo se mitigaría**: esto no se "parchea", porque no es un
fallo de software corregible: si hay una shell abierta sin
autenticación, el sistema está comprometido. Las acciones a tomar son:
aislar la máquina de la red, identificar el proceso que abre ese
puerto y el binario asociado, reinstalar el sistema desde cero con
medios verificados y realizar una auditoría forense para determinar el
alcance del compromiso. A futuro: no instalar binarios de fuentes no
oficiales, verificar firmas y hashes, y monitorizar los puertos
abiertos.

**Referencia**: Plugin ID de Nessus **51988**, familia *Backdoors*.
Sin CVE asociado.

## 6. Recomendaciones

Ordenadas por prioridad:

1. **Aislar y reinstalar el sistema afectado por la backdoor.** No se
   puede confiar en un sistema donde se ha detectado una shell sin
   autenticación: hay que reinstalar desde cero y auditar cómo se
   introdujo esa puerta trasera.
2. **Corregir las configuraciones débiles de los servicios expuestos.**
   Cambiar contraseñas por defecto (VNC), deshabilitar servicios
   innecesarios (FTP anónimo, MySQL sin contraseña, etc.) y restringir
   el acceso a los servicios por firewall.
3. **Deshabilitar protocolos obsoletos.** Eliminar SSL 2.0 y 3.0,
   dejando únicamente TLS 1.2 o superior con conjuntos de cifrado
   aprobados.
4. **Establecer un plan de actualización y auditoría periódica.** No
   volver a dejar sistemas sin revisar durante años: parcheo regular,
   inventario de servicios, escaneos programados y política de
   contraseñas.

## 7. Conclusión

A la vista de los resultados, **no firmaría el contrato de mantenimiento
en las condiciones actuales**. La presencia de una backdoor activa,
protocolos obsoletos y contraseñas por defecto indica que los sistemas
no han recibido ningún tipo de mantenimiento de seguridad, y que
cualquiera con conocimientos básicos podría comprometerlos.

Firmaría el contrato **con condiciones**:

- Compromiso de reinstalación y securización inicial de los sistemas
  antes de la firma.
- Plan de actualizaciones y auditorías periódicas documentado.
- Política de contraseñas y de accesos mínimos.
- Monitorización continua de puertos y servicios expuestos.
