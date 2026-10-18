# Parte B · Clasificar amenazas

| # | Incidente en Torrent | Tipo | ¿Se replica solo? | ¿Necesita al usuario? |
|---:|---|---|---|---|
| 1 | Un socio abre `factura_marzo.pdf.exe`; poco después todo el disco compartido está cifrado y aparece un mensaje pidiendo 800 €. | Ransomware | puede propagarse a otros equipos, pero no es su característica principal, no necesariamente se replica | Sí, para ejecutar el archivo inicialmente |
| 2 | Un "optimizador gratuito" de PC que, además, envía todo lo que se teclea a un servidor externo. | Troyano y keylogger | No | Sí, para instalarlo y ejecutarlo |
| 3 | Sin que nadie haga nada, un programa salta de portátil en portátil a través de la wifi del estudio. | Gusano | Sí | No, una vez iniciado, puede propagarse automáticamente |
| 4 | Al abrir ciertos ejecutables, estos infectan a su vez a otros ficheros del equipo. | Virus | Sí, infecta otros archivos, aunque necesita ejecutarse mediante un archivo infectado. | Sí, para ejecutar el archivo infectado inicialmente |
| 5 | Tras instalar un editor de PDF gratuito, aparecen ventanas de publicidad constantemente. | Adware | No | Sí, para instalarlo normalmente |
| 6 | Un antiguo colaborador dejó un código que borrará los planos el 31 de diciembre. | Bomba lógica | No | No en el momento de activarse, se ejecuta cuando se cumple la condición programada |
| 7 | El ordenador del estudio está enviando spam sin que nadie lo sepa, bajo las órdenes de un servidor externo. | Bot/zombi | No necesariamente, puede formar parte de una red de bots controlada remotamente. | No para enviar spam una vez comprometido el equipo |
| 8 | Un conjunto de programas modificados que esconde procesos y ficheros del intruso dentro del propio sistema. | Rootkit | No necesariamente | No para ocultarse una vez instalado |
| 9 | Un servicio escuchando en un puerto que permite entrar al equipo sin contraseña. | Backdoor (puerta trasera) | No | No si el atacante puede acceder directamente por esa puerta trasera |
| 10 | Correo: "Tu banco necesita que confirmes tus datos en este enlace". | Phishing | No | Sí, para caer en el engaño y facilitar los datos |
| 11 | Mensaje: "Reenvía esto a 10 contactos o tu cuenta se cerrará". | Hoax (bulo) | No | Sí, para creerlo y difundirlo |
| 12 | Una web idéntica a la del ERP a la que se llega porque el DNS ha sido manipulado. | Pharming | No. | No necesariamente; la redirección puede producirse automáticamente al visitar la web |
| 13 | Un programa que registra sin permiso los hábitos de navegación y los vende. | Spyware | No necesariamente | Sí, normalmente para instalarlo inicialmente, aunque puede hacerlo otro programa sin su conocimiento |

## Pregunta A · Fraude e ingeniería social

el caso 10 y 11 son por ingenieria social, el phishing engaña a la persona al igual que el Hoax. un antivirus no los podria deectar al no tener codigo malicioso involucrado en este caso si no redirigir a la persona a otra pagina con la confianza de la identidad del remitente

El pharming (caso 12) también puede redirigir a una víctima sin infectar directamente su dispositivo con malware, mediante la manipulación del DNS. Por eso un antivirus por sí solo no siempre puede evitarlo.
