## Paso 1. Datos encontrados 

|   Dato encontrado | Fuente (1-6) | ¿Para que le sirve al atacante? | Contramedida propuesta |
| -------: | ----------- | ----------------------------------- | ----------------- |
| Correos corporativos y nombres de empleados: mferrer@, avidal@, lperis@ rrhh@ | 1, 2, 6  | Permiten phishing dirigido, spar-phshing y enumeracion de cuentas. se sabe quien es quien | Politica de no publicar correos nominales; usar formularios de contado, formacion anti phishing y autenticacion de doble factor |    
| El ERP es GestiCloud en la nube y el portal de clientes esta en portal.torrent-estudio.example | 1, 4 | Identifica el proveedor SaaS y el subdominio del portal. se busacrian vulnerabilidades en gestiCloud, credenciales filtradas o ataques de sesion contra el portal | Auditar el proveedor, exigir MFA, revisar permisos, monitorizar acceso al portal |
| Servidor Ubuntu 8.04 con Apache y vsftpd | 2 | Ubuntu 8.04 esta obsoleto y sin soporte, los servicios antiguos tienen CVEs conocidos (vulnerablidades comunes) | Actualizar a una version soportada, segmentar el servidor, cerrar FTP y usar en cambio SFTP/HTTPS, firewwal y parcheo continuo |
| NAS con copias de seguridad | 2 | Objetivo clasico de un ransomware, en caso de que lo cifren la empresa no lo puede recuperar | copias 3-2-1, NAS fuera de dominio, credenciales diferentes, cifrado y pruebas de restauracion |
| 3 pusetos con windows 7 | 2 | sistema sin soporte, vulnerable a exploits conocidos. facilita el movimiento lateral y randomware | cambairse a windows 10 u 11 soportado, parches, bloqueo de macros y USB |
| servidor de impresion en un portatl y red wifi de oficina | 2 | un portatil como servidor de impresion es un riesgo, el wifi puede ser atacado por proximidad (evil twin, deauth, cracking WPA) | segmentar red, AP aislado, equipo dedicado y actualizado |
| WHOIS con nombre real, direccion, telefono y correo personal de Marta Ferrer | 3 | Ingenieria social, llamar haciendose pasar por un cliente, obtener mas datos, recuperar contraseñas o hacer vishing. el correo personal puede estar en filtraciones | privacidad WHOIS, correo generico, no usar datos personales, formacion en vishing |
| registros DNS: nas, pruebas, mail, erp y portal con IPs publicas | 4 | pruebas suele ser un entorno debil, nas expuesto es importante, mail permite atacar el correo, le estan dando un roadmap a los atacantes | minimizar los registros publicos, no exponer NAS ni entornos de pruebas, usar VPN, revisar DNS |
| Publicacion con foto donde se ve una nota adhesiva amarilla con texto legible | 5 | Seguridad fisica y visual: la nota puede contener contraseñas, usuarios o indicaciones internas, ademas de que tambien revela equipos, marca Canon y distribucion | Politica de no publicar fotos en los puestos de trabajo, borrar notas, limpiar mesas, formacion sobre esto |
| Marta teletrabaja en cafeteria con wifi publico | 5 | ataques de red local como sniffing, evil twin, robo de sesion. tambien revela horarios y ubicaciones | VPN coporativa obligatoria, no usar wifi publico sin esto y evitar informacion sensible en publico |
|horario laboral y cierre de agosto | 1, 6 | Planificar ataques fuera de horario o en periodo de baja vigilancia | no publicar cierres detalladas, monitorizacion 24/7 o externalizada |
|SPF v=spf1 +all y ausencia de DKIM/DMARC | 4 | permite spoofing de correo, enviar emails falsos como si fueran torrent-estudio.example. ideal para phishing a clientes o empleados | configurar SPF estricto (-all), DKIM y DMARC con politica reject |


## Paso 2. Plan de ataque
aprovecharia el SPF +all y la falta de DMARC para enviar correos suplantandoa Lucía Peris y solicitar un cambio de datos bancarios, coincidiendo con agosto. A la par, lanzaria spear-phishing a Marta y Andrés con enalce a un portal falso mientras se escanea pruebas y nas por servicios obsoletos. bucaria exploits publicos para los sistemas antiguos y obtener acceso unicial. una vez adentro se cifra el NAS y pediria el rescate. Todo en base a llamadas de vishing usando los telefonos y nombres obtenidos.

## Paso 3. Las 3 contramedidas mas importantes

1. configurar SPF estricto, DKIM y DMARC con politica reject.
Seria la mas critica porque el correo es la entrada principal. Sin DMARC cualquiera puede suplantar a lucia, marta o andres y engañar a clientes o empleados. Bloquea phishing, frauda del CEO y spoofing de forma directa.
2. Actualizar y segmentar la infraestructura obsoleta (Ubuntu 8.04, windows 7, ftp, NAS expuesto).
Aunque la ingenieria social es muy buena, los sistemas sin soporte tiene exploits publicos que ptermiten e acceso sin que alguien mas se involucre.
3. Fromacion anti-phishing y procedimientos de verificacion para pagos y cambios de datos bancarios.
la tecnologia sola no basta, la respuesta automatica y las publicaciones ya dan bastante para atacar. un empleado formado que verifique por segundo canal evitaria el fraude aunque el correo parezca legitimo. esto protege lo que el firewall no ve.
