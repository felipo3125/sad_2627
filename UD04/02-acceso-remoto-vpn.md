# Práctica: VPN, certificados y seguridad de la red

## Parte 0: La red del laboratorio

En esta primera parte configuramos las interfaces de red en VirtualBox y
asignamos las IP estáticas para aislar la red interna (`net-lan`) de la
externa (`net-externa`). Antes de montar la VPN, comprobamos que el
cliente externo no tiene conectividad con la planta (HMI).

### 0.4 Prueba sin VPN (Entrega 0)

**Comprobación:** Con el adaptador NAT desconectado, el cliente intenta
hacer ping al HMI y falla, demostrando que sin túnel no hay acceso.

<img width="955" height="915" alt="host inalcanzable cable descoenctado para LAN" src="https://github.com/user-attachments/assets/7aa8db79-aa55-44e7-9d6b-06e503787b6d" />


------------------------------------------------------------------------

## Parte 1: Montar la CA con Easy-RSA y emitir certificados

Creamos la Autoridad de Certificación (CA) en `srv-ca` con Easy-RSA.
Generamos las peticiones de firma (CSR) en la pasarela y en el cliente,
y la CA las firma con los roles correspondientes (`serverAuth` y
`clientAuth`).

### Entrega 1

**Comprobación:** Verificamos que los certificados están correctamente
firmados y que cada uno tiene su rol (`TLS Web Server Authentication` y
`TLS Web Client Authentication`). El archivo `index.txt` refleja ambos
certificados como válidos (`V`).

**1. Creación de la CA y verificación:**
<img width="1280" height="800" alt="open ssl en srv ca completo" src="https://github.com/user-attachments/assets/bfca5fe7-d717-4b10-9a8c-e69fd5e2a72a" />

**2. Firma del certificado de la pasarela (vpn-gw) con su SAN:**
<img width="1280" height="800" alt="primer certificado firmado" src="https://github.com/user-attachments/assets/837b9d7e-f8c9-47b3-abac-014e99b037b3" />


**3. Firma del certificado del técnico (tecnico01):**

<img width="1280" height="800" alt="certificado de tecnico01 firmado y verificado" src="https://github.com/user-attachments/assets/1dd86624-4358-47df-8c0e-d2c854d9b5cb" />


**4. Generación de la Lista de Revocación (CRL):**

<img width="1280" height="800" alt="crear archivo punto pem para la pasarela (mas adelante)" src="https://github.com/user-attachments/assets/4789029c-f12b-4c80-80e8-da817926601a" />


**Diferencias entre `x509-types/server` y `x509-types/client`:**

El certificado del servidor incluye el uso `serverAuth`, mientras que el
del cliente incluye `clientAuth`. Esta distinción permite comprobar que
cada equipo utiliza un certificado con el rol que le corresponde y ayuda
a evitar suplantaciones.

------------------------------------------------------------------------

## Parte 2: La pasarela `dmz-vpn`

Configuramos OpenVPN en la pasarela, activando el reenvío de paquetes
(`ip_forward`), colocando los certificados y la CRL, y protegiendo el
canal de control con `ta.key`.

### Entrega 2

**Comprobación:** El servicio OpenVPN está activo (`running`), la
interfaz del túnel `tun0` se ha creado con la IP `10.8.0.1` y el puerto
UDP 1194 está a la escucha.

<img width="1280" height="800" alt="primera captura para habilitar y encender openvp" src="https://github.com/user-attachments/assets/d1497d68-ab54-4b52-a0e0-ea19e225e2ca" />


------------------------------------------------------------------------

## Parte 3: Cortafuegos mínimo en la pasarela

Aplicamos una política restrictiva con `nftables` para que, aunque
alguien entre por la VPN, solo pueda llegar al HMI por SSH y hacer ping.
Todo lo demás se descarta.

### Entrega 3

**Comprobación:** Las reglas de `nftables` muestran la política `drop`
por defecto y solo permiten el tráfico explícito hacia `10.0.20.20`.

<img width="1280" height="800" alt="activar la configuracion verificar sintaxis y mostrar lista ruleset" src="https://github.com/user-attachments/assets/719325d5-54bb-4ded-8bd3-355fb953e0e7" />


------------------------------------------------------------------------

## Parte 4 y 5: El cliente y comprobación del túnel

Configuramos el cliente con su certificado, el archivo `ta.key` y el
archivo `cliente.conf`. Nos conectamos y comprobamos que el túnel se
levanta y que tenemos conectividad con el HMI.

### Entrega 4

**Comprobación:** El túnel se levanta correctamente
(`Initialization Sequence Completed`), la ruta a la red `10.0.20.0/24`
apunta a `tun0` y el ping al HMI responde. Además, se puede acceder por
SSH al HMI.

**1. Conexión del túnel establecida:**

<img width="1280" height="800" alt="tunel establecido correctamente" src="https://github.com/user-attachments/assets/46b921ec-5f05-4c0c-8521-251991b11951" />

**2. Comprobación de conectividad (Ping a la pasarela y al HMI):**

<img width="1280" height="800" alt="ping final para comprobar que hay comunicacion" src="https://github.com/user-attachments/assets/b4245bfe-7ff5-4857-9060-11f98f3bd3cd" />

**3. Acceso por SSH al HMI a través de la VPN:**

<img width="1280" height="800" alt="salida ssh a la pasarela de hmi" src="https://github.com/user-attachments/assets/fff4bc96-c1da-4787-9a7d-33a315414b7c" />


------------------------------------------------------------------------

## Parte 6: Pruebas de ataque y defensa

### 6.1 La ruta no es un control de acceso (Entrega 5)

**Comprobación:** El cliente puede llegar al HMI, pero no a la CA (`10.0.20.30`), que aparece como `filtered`. Tampoco puede llegar a redes inventadas (`192.168.99.1`).

**Explicación:** El cliente llega al HMI porque la regla `forward` de nftables permite explícitamente ese tráfico. No llega a la CA porque no existe ninguna regla que autorice ese tráfico y la política por defecto es `drop`.

<img width="1280" height="800" alt="confirmar que la pasarela no deja pasar a cualquiera y el hmi tiene el puerto abierto junto con direccion inventada" src="https://github.com/user-attachments/assets/8c4f3be8-91cd-47f7-8d60-94795bec8cb0" />


### 6.2 Qué ve un atacante en el cable (Entrega 6)

**Comprobación:** En la interfaz externa (`enp0s8`) solo se ve tráfico UDP 1194 cifrado. En la interfaz `tun0` se ve el tráfico ICMP hacia `10.0.20.20` ya descifrado.

**Explicación:** Esto demuestra la confidencialidad del túnel. Un atacante que observe el tráfico desde el exterior solo verá los paquetes UDP cifrados, mientras que dentro del túnel circula el tráfico real.

<img width="1280" height="800" alt="capturar dentro del tunel al desencriptar el trafico" src="https://github.com/user-attachments/assets/bdaa9b67-6bca-4132-bab0-1c30d7498a74" />

*(En esta captura se observa el tráfico externo, donde solo se ven las IPs públicas y el puerto 1194, sin rastro de las IPs internas de la planta).*
<img width="1280" height="800" alt="trafico de la pasarela en dmz de la ip del auditor" src="https://github.com/user-attachments/assets/6a0dd4c8-84d2-42c8-a7af-fdc9ee204acc" />

*(En esta captura se observa el tráfico dentro de `tun0`, donde ya se ve el ping descifrado hacia el HMI).*

### 6.3 Revocación de un certificado (Entrega 7)

**Comprobación:** Tras revocar el certificado en la CA y actualizar la CRL en la pasarela, el cliente ya no puede conectar. El registro de la pasarela muestra `VERIFY ERROR: error=certificate revoked`.

**1. Proceso de revocación en la CA:**
<img width="1280" height="800" alt="revocar certificado al tecnico al simular un robo" src="https://github.com/user-attachments/assets/0013ab45-9d23-4da4-ab18-803d1015f72e" />


**2. Verificación de que el certificado está revocado:**
<img width="1280" height="800" alt="permiso revocado exitoso" src="https://github.com/user-attachments/assets/21d62f59-8de1-4e1f-a865-c6606ec6a621" />


**3. Registro de la pasarela rechazando la conexión:**

<img width="1280" height="800" alt="confirmar que la pasarela no deja pasar a cualquiera y el hmi tiene el puerto abierto junto con direccion inventada" src="https://github.com/user-attachments/assets/d112c0a8-c87d-4cf9-a2f6-1d2c64dc8326" />


------------------------------------------------------------------------

## Parte 7: Preguntas de reflexión

### 1. Un atacante roba `tecnico01.key`. ¿Qué puede hacer y qué le falta para entrar?

El atacante tendría la clave privada del técnico, por lo que podría
intentar autenticarse haciéndose pasar por él. También necesitaría los
demás elementos requeridos por la configuración, como el certificado
correspondiente y la clave compartida `ta.key`. No necesita la
contraseña de la CA para conectarse: esa contraseña protege la clave
privada de la CA, no la autenticación habitual del cliente. Si se
detecta el robo, se puede revocar el certificado y actualizar la CRL
para que la pasarela lo rechace.

### 2. ¿Por qué `ca.key` no sale nunca de `srv-ca`? ¿Qué podría hacer alguien que la robara?

Porque es la clave privada principal de la Autoridad de Certificación.
Si alguien la robara, podría emitir y firmar certificados falsos que
parecerían legítimos para los equipos que confían en esa CA. Esto
pondría en riesgo la seguridad de la VPN, por lo que la CA se mantiene
apagada y aislada la mayor parte del tiempo.

### 3. ¿Qué ocurre si la CRL caduca y nadie la renueva?

Si la CRL caduca, OpenVPN puede rechazar las conexiones porque ya no
puede comprobar de forma válida el estado de revocación de los
certificados. Esto puede impedir la conexión incluso a clientes que
tengan certificados válidos, dejando el acceso VPN interrumpido hasta
que se genere y se instale una CRL vigente.

### 4. La ruta del túnel anunciaba toda la LAN. ¿Por qué eso no era un problema? ¿Qué lo habría convertido en uno?

No era un problema porque la ruta solo indica por dónde enviar los
paquetes; no concede permisos de acceso. Quien decide si el tráfico
puede pasar es el cortafuegos de la pasarela. Se habría convertido en un
problema si el cortafuegos estuviera mal configurado, permitiera más
tráfico del necesario o tuviera una política por defecto de aceptación
(`accept`).

### 5. Si cae la propia pasarela, ¿qué consigue el atacante y qué lo frena? Relaciona la respuesta con la DMZ y con los flujos que definiste en la Práctica 1.

Si un atacante compromete la pasarela, obtiene el control del equipo que
conecta la red externa con la LAN y podría intentar modificar sus reglas
o generar tráfico hacia la red interna. La DMZ y la segmentación limitan
la exposición de la LAN y mantienen separados los servicios, mientras
que los flujos definidos en la Práctica 1 restringen las comunicaciones
permitidas. Sin embargo, esas reglas no garantizan por sí solas la
protección si la propia pasarela queda completamente comprometida.

### 6. Si firmaras el certificado del técnico con `sign-req server` en lugar de `sign-req client`, ¿qué podría hacer ese técnico que ahora no puede? ¿Qué mecanismo de OpenVPN lo impediría?

El técnico tendría un certificado con el uso `serverAuth` en lugar de
`clientAuth`, por lo que podría utilizarlo para poner en marcha un
servidor OpenVPN que se hiciera pasar por uno legítimo. Si otros
clientes se conectaran a ese servidor, podría intentar interceptar sus
comunicaciones. La opción `remote-cert-tls server` en los clientes
comprueba que el certificado del servidor tenga el uso adecuado,
mientras que `remote-cert-tls client` en la pasarela exige certificados
de cliente al autenticar a quienes se conectan.
