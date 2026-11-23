# Práctica 1 · Perímetro y zonas

## 1. Tabla de ubicación

| Servicio | Zona | Justificación |
|---|---|---|
| Web corporativa pública | DMZ | Debe ser accesible desde Internet, pero sin acceso directo a la red interna. |
| Servidor de correo | DMZ | Necesita comunicarse con servidores externos para enviar y recibir correos. |
| Pasarela VPN | DMZ | Permite las conexiones remotas de usuarios autorizados. |
| Proxy (saliente e inverso) | LAN / DMZ | El saliente controla la navegación interna y el inverso recibe conexiones externas. |
| HMI de planta y carpeta de planos | Zona crítica | Contienen información importante para la empresa y necesitan protección. |
| Autoridad de certificación (srv-ca) | Zona crítica | Gestiona certificados y claves que deben mantenerse protegidos. |
| Servidor de copias de seguridad | Zona crítica | Guarda copias de información importante que no deben quedar expuestas. |
| Portátiles de ingenieros y operarios | LAN | Son equipos propios de los trabajadores. |
| Wi-Fi de invitados | Zona de invitados | Separa los dispositivos externos de la red de la empresa. |
| Servidor de registros y monitorización | Zona crítica | Guarda información sobre la actividad de la red y los equipos. |

## 2. Tabla de flujos

| Origen → destino | ¿Permitido? | Ejemplo concreto |
|---|---|---|
| Internet → DMZ | Sí, limitado | Acceso a la web mediante HTTPS y a la pasarela VPN. |
| Internet → LAN | No | No se permiten conexiones directas desde Internet. |
| DMZ → LAN | Solo en casos necesarios | La VPN permite acceder al HMI a usuarios autorizados. |
| LAN → DMZ | Sí, controlado | Administración de la web mediante SSH. |
| LAN → Internet | Sí | Los trabajadores pueden navegar utilizando el proxy. |

La red de invitados solo tendrá acceso a Internet, sin poder comunicarse con las demás zonas

## 3. Diagrama de la red

<img width="3066" height="1116" alt="mermaid_diagrama_hecho" src="https://github.com/user-attachments/assets/7e2196f8-cc8c-4450-bd3d-2e8d5ae92c76" />


## 4. Pregunta de reflexión

Si alguien consigue entrar en el servidor web podría controlarlo e intentar acceder a otros equipos. pero, al estar en la DMZ, no tendría acceso directo a la LAN ni a la zona crítica. Los cortafuegos impiden esas conexiones y limitan lo que puede hacer el atacante.
