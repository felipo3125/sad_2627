# Ataques de suplantación de identidad

## 1. SMTP spoofing

### Qué es
El email spoofing, o correo de suplantación de identidad, es una técnica empleada en ataques de spam y phishing para hacerle creer a un usuario que un mensaje proviene de una persona o entidad que conoce o en la que confía. El atacante falsifica los encabezados del correo electrónico para que el cliente de correo muestre una identidad falsa. Si el nombre es reconocido, es más probable que la víctima confíe, abra archivos adjuntos con malware, envíe datos sensibles o incluso realice transferencias de dinero.
### Cómo se lleva a cabo
El protocolo SMTP (Simple Mail Transfer Protocol) fue diseñado originalmente sin mecanismos de autenticación de identidad. Cualquier usuario o servidor puede modificar libremente campos del encabezado como el From: (De:). Esto equivale a escribir cualquier nombre o dirección falsa en el remitente de un sobre de carta física antes de echarlo al buzón. La aplicación cliente asigna una dirección de remitente a los mensajes salientes, por lo que los servidores de correo saliente no pueden identificar si la dirección del remitente es legítima o falsa.


### Qué categoría(s) de amenaza compromete
Autenticidad: se compromete de manera crítica al falsificar la identidad del remitente.

Integridad: se vulnera al alterar los encabezados del correo.

Confidencialidad: se ve afectada de forma indirecta mediante técnicas de phishing.

Disponibilidad: no aplica.

### Ejemplo o caso real
El caso real más impactante es el fraude de 100 millones de dólares a Google y Facebook (2013-2015). Un atacante usó SMTP spoofing para enviar correos electrónicos falsos que suplantaban a Quanta Computer, un proveedor legítimo de hardware de ambas empresas. Los correos incluían facturas falsas que parecían 100% reales. Los departamentos financieros confiaron en la identidad del remitente y transfirieron el dinero a cuentas del estafador antes de ser descubiertos.

### Medida de prevención
Hoy en día, la seguridad del correo electrónico se basa en tres protocolos de autenticación que trabajan juntos:

SPF (Sender Policy Framework): registro DNS donde el dueño de un dominio publica una lista de direcciones IP autorizadas para enviar correos en su nombre.

DKIM (DomainKeys Identified Mail): añade una firma criptográfica digital a los encabezados. El servidor destinatario usa la clave pública del remitente (alojada en DNS) para verificar que el mensaje se originó en ese dominio y no fue alterado.

DMARC (Domain-based Message Authentication, Reporting, and Conformance): vincula las verificaciones de SPF y DKIM. Permite al dueño del dominio indicar a los servidores destinatarios qué hacer si un correo falla la autenticación (por ejemplo, p=reject o p=quarantine).

### Fuente
Ejemplo real:
https://www.kaseya.com/es-la/blog/worst-phishing-attacks-in-history/
https://www.kaseya.com/es-la/blog/worst-phishing-attacks-in-history/
https://www.welivesecurity.com/es/seguridad-corporativa/que-es-spoofing-de-email/

## 2. DNS spoofing

### Qué es
Ciberataque donde se alteran los registros de un servidor o caché DNS para redirigir a los usuarios a páginas maliciosas. Se interceptan los datos de los usuarios al introducirlos en la caché DNS para usarlos.
### Cómo se lleva a cabo
Alterando los registros del sistema de nombres de dominio para redirigir el tráfico de un usuario hacia una dirección IP fraudulenta y maliciosa.
### Qué categoría(s) de amenaza compromete
Confidencialidad: el atacante puede ver la información de los usuarios.

Integridad: al alterar los nombres de dominio.

Disponibilidad: al no estar accediendo al sitio web deseado.
### Ejemplo o caso real
En 2015, hackers realizaron un ataque DNS contra una aerolínea (Malaysia Airlines). Al acceder a la página web salía un error 404 y una foto de un lagarto. Vulneraron el DNS, pero se desconoce si vulneraron los datos de los usuarios.
### Medida de prevención
Usar HTTPS: cifra la comunicación y ayuda a detectar redirecciones maliciosas mediante certificados SSL.
Usar resolutores DNS confiables (Cloudflare 1.1.1.1, Google 8.8.8.8, Quad9).
Verificar la configuración DNS del router y cambiar contraseñas por defecto.
Limpiar la caché DNS si se sospecha un ataque.
### Fuente
grupo 2

## 3. IP spoofing

### Qué es
El IP spoofing es una técnica donde el atacante modifica el campo de dirección de origen en la cabecera de un paquete de red, haciéndolo parecer que proviene de una dirección IP diferente (inventada o de la propia víctima).

### Cómo se lleva a cabo
El atacante modifica la dirección de origen de la cabecera del paquete que está enviando o interceptando, haciendo parecer que se envía desde otro lugar que se supone de confianza. Al cambiar la dirección de origen, el paquete puede pasar controles de acceso basados en IP, y si el atacante no necesita recibir la respuesta, puede completar acciones sin ser detectado directamente.

### Qué categoría(s) de amenaza compromete

Autenticidad: se rompe al falsificar la dirección de origen, haciendo creer que el paquete proviene de una fuente legítima.
Integridad: el atacante puede modificar el contenido del paquete.
Disponibilidad: también puede verse afectada.
Confidencialidad: puede verse comprometida indirectamente.

### Ejemplo o caso real
El caso más famoso es el ataque de Kevin Mitnick a Tsutomu Shimomura en diciembre de 1994. Mitnick utilizó IP spoofing y predicción de números de secuencia TCP para hacerse pasar por una máquina de confianza y obtener acceso a una estación de trabajo de Shimomura. El ataque comenzó el 25 de diciembre de 1994 a las 14:09:32 PST. Mitnick falsificó la dirección IP de un host de confianza (server.login) para engañar al sistema objetivo y obtener acceso sin contraseña.
### Medida de prevención
Filtrado de paquetes: configurar routers y firewalls para rechazar paquetes entrantes que afirmen tener una dirección de origen perteneciente a la red interna. Esto se conoce como filtrado de entrada (ingress filtering).

Infraestructura de clave pública (PKI): usar autenticación criptográfica para verificar identidades.

Formación en seguridad: capacitar a los usuarios para que no caigan en enlaces sospechosos y reconozcan intentos de phishing.

### Fuente
incibe
grupo 1

## 4. Captura de cuentas de usuario y contraseñas

### Qué es
La captura de cuentas de usuario y contraseñas es una táctica de ciberataque centrada en el robo de credenciales (contraseñas, hashes, tokens y cookies) para lograr persistencia y movimiento lateral en sistemas comprometidos.

### Cómo se lleva a cabo
Malware infostealer: software malicioso que se instala en dispositivos y extrae contraseñas, tokens y cookies almacenados en navegadores.
Phishing: engañar a los usuarios para que introduzcan sus credenciales en sitios falsos.
Credential stuffing: usar credenciales filtradas de una brecha para intentar acceder a otras cuentas donde el usuario reutilizó la misma contraseña
### Qué categoría(s) de amenaza compromete
Confidencialidad: se compromete directamente al robar credenciales y datos de autenticación.

Integridad: el atacante puede modificar datos usando las cuentas comprometidas.

Autenticidad: se suplanta la identidad del usuario legítimo.

Disponibilidad: el usuario legítimo puede perder acceso a su cuenta.
### Ejemplo o caso real
En junio de 2025, se descubrió una de las mayores filtraciones de credenciales de la historia: aproximadamente 16 mil millones de credenciales de inicio de sesión quedaron expuestas en bases de datos sin protección. Los datos incluían accesos a Apple, Google, Facebook, Telegram y VPNs, entre otros. 
### Medida de prevención
Usar contraseñas seguras y únicas para cada cuenta, evitando la reutilización.

Activar autenticación multifactor (MFA), preferiblemente basada en aplicaciones (no solo SMS) .

Usar gestores de contraseñas y adoptar Passkeys donde sea posible .

Monitorear alertas en Dark Web y proteger entornos cloud contra configuraciones inseguras .

Implementar monitoreo de seguridad continuo para detectar actividades inusuales

### Fuente
grupo 3
grupo
## Aplicado a Estudio Torrent
la mas plausible seria SMTP spoofing y en seungo lugar captura de cuentas y contraseñas de usuarios
y creemos esto porque estudio Torrent rabaja con clientes y proveedores por correo, por lo que un atacante podria suplantar la identidad de un liente o compañero para solicitar facturas, transferencias o acceso al disco compartido por los clientes.
