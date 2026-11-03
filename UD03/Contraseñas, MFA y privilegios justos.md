# Práctica: Contraseñas, MFA y Privilegios Justos (Caso Torrent S.L.)

la guía documenta la configuración de una política de contraseñas robusta, implementacion de doble factor (MFA) para SSH y la delegacion de privilegios mediante el comando sudo

## 1. Contraseñas robustas con PAM
el objetivo es forzar una politica de contraseña segura a nivel de sistemas para todos los usuarios

### 1.1 instalacion y activacion de pwquality
instalamos el modulo de PAM que evalua la calidad de contraseñas
```bash
sudo apt update
sudo apt install libpam-pwquality
```
verificamos que la regla este activa en el archivo de configuración de PAM. Deberia aparecer una linea con pam_pwquality.so justo antes de pam_unix.so

```bash
grep -n pwquality /etc/pam.d/common-password
```
si no aparece se debe añadir manualmente

```bash
password requisite pam_pwquality.so retry=3
```

### 1.2 configuracion de la politica de contraseñas
editamos el archivo de confiuracion para exigir contraseñas mas potentes

```bash
sudo nano /etc/security/pwquality.conf

#en el archivo

minlen = 12
minclass = 3
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
```
exigimos mínimo de 12 caracteres, al menos 3 tipos diferentes de caracteres y obligamos a usar al menos un digito, una mayúscula, una minuscula y un simbolo

### 1.3 gestion de la caducidad con chage
forzamos a que las contraseñas caduquen periodicamente para limitar la ventana de exposicion en caso de filtracion

```bash
sudo chage -M 90 -m 1 -W 7 alberto
sudo chage -l alberto
```
-M 90 (caduca en 90 dias)
-m 1 (minimo 1 dia para poder cambiarla)
-W 7 (avisa 7 dias antes de que expire)

## 2. autenticacion de doble factor (MFA/TOTP) para SSH 

### 2.1 instalacion del modulo PAM
```bash
sudo apt install libpam-google-authenticator
```

### 2.2 generacion del secreto (por cada usuario)
cada usuario debe ejecutar este comando en su propia cuenta para vinular su aplicacion de telefono (Google Authenticator, Authy, etc)
```bash
google-authenticator
```
respuestas recomendadas para el asistente:
* Tokens time-based? -> Y
* actualizar archivo? -> Y
* impedir mismo código dos veces? -> Y
* ampliar ventana de validez -> N
* limitar intentos (rate-limit)? -> Y

nota: escanear el código QR con el teléfono y guardar los 5 códigos de emergencia en un lugar seguro

### 2.3 integracion con SSH
editamos la configuracion de PAM para SSH

```bash
sudo nano /etc/pam.d/sshd

#añadimos al final del archivo despues de @include common-auth

auth required pam_google_authenticator.so
```

editamos la configuracion del servicio SSH

```bash
sudo nano /etc/ssh/sshd_config

#aseguramos que estas directivas esten configuradas de la siguiente forma

#KbdInteractiveAuthentication yes
#UsePAM yes
#PasswordAuthentication yes
```

reiniciamos el servicio para aplicar los cambios

```bash
sudo systemctl restart ssh
```


### 2.4 exigir MFA solo a un grupo especifico (con pam_succeed_if)
para no obligar a todos a usar MFA creamos un grupo y conficionamos la regla de PAM

crear grupo y añadir a los usuarios correspondientes
```bash
sudo groupadd mfa_obligatorio
sudo usermod -aG mfa_obligatorio carlos

#modificamos /etc/pam.d/sshd en lugar de la regla anterior, ponemos estas dos lineas

#auth [success=1 default=ignore] pam_succeed_if.so user notingroup mfa_obligatorio
#auth required pam_google_authenticator.so
```


## 3. delegacion de privilegios con sudo 
el objetivo es evitar el uso de la cuenta root y dar solo los permisos necesarios

### 3.1 creacion del script de auditoria

```bash
sudo tee /usr/local/bin/auditoria_planta.sh << 'EOF'
#!/bin/bash
echo "[$(date)] Auditoria de planta ejecutada por $(whoami)"
EOF
sudo chmod 755 /usr/local/bin/auditoria_planta.sh
```

### 3.2 configuracion de reglas con sudoers
usamos visudo (valida la sintaxis) para crear un archivo especifico

```bash
sudo visudo -f /etc/sudoers.d/torrent

#añadimos las siguientes reglas  (el grupo operarios no tiene permisos)

#%ingenieria ALL=(root) /usr/bin/systemctl restart hmi_planta, /usr/bin/systemctl status hmi_planta
#%supervisores ALL=(root) NOPASSWD: /usr/local/bin/auditoria_planta.sh
```

explicacion
%ingenieria puede reiniciar y ver el estado del servicio, pero se le exige su contraseña

%supervisores puede ejecutar el script de auditoria sin pedir contraseña (NOPASSWD), ideal para tareas automatizadas o scripts de bajo riesgo

### 3.3 verificacion de privilegios
comprobamos que puede hacer cada usuario

```bash
sudo -l -U nestor
sudo -l -U carlos
sudo -l -U alberto
```

## preguntas de reflexion

### 1. ¿Qué riesgo cubre el MFA que una contraseña de 20 caracteres no cubre?

una contraseña fuerte puede proteger contra fuerza bruta pero no controla el robo de credenciales como lo puede ser un phishing, keylogers, filtraciones de base de datos. El MFA añade una capa de algo que tiene (el secreto del telefono), por lo que si un atacante obtiene la contraseña seguira necesitando el codigo dinamico para poder acceder

### 2. Postura del NIST sobre la caducidad obligatoria de contraseñas.
El NIST desaconseja forzar el cambio de contraseñas de forma periódicas (cada 90 días por ejemplo) a menos que haya indicios de compromiso. obligar cambios frecuentes provoca que los usuarios elijan contraseñas predecibles o debiles ya sea cambiando una leta o un numero, o que las apunten en lugares inseguros, lo que reduce la seguridad real

### 3. porque sudo no sustitye a las ACL?
son capas complementarias. sudo controla que comandos puede ejecutar un usuario con privilegios de root. las ACL controlan que archivos puede leer o escribir un usuario con su propia identidad. un ejemplo seria que un ingenierio puede tener permiso sudo para reiniciar la base de datos, pero si las ACL del directorio de planos no le dan permiso de lectura entonces no podra leer los planos y ni si quiera usando el comando sudo 

### 4. perdido del telefono sin scratch codes 
el administrador del sistema (con acceso root o sudo) debe eliminar o regenerar el archivo de configuracion del MFA del usuario afectado

```bash
sudo rm /home/usuario/.google_authenticator
```
posteriormente, el suuario debera ejecutar de nuevo google-authenticator para vincular un nuevo dispositivo. el resto de cuentas no estarian afectadas ya que el secreto TOTP es unico por usuario

