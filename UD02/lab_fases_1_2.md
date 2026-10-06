# Laboratorio fases 1 y 2 — Estudio Torrent

## A1. whois y nslookup

No es necesario. Este punto se trabajó sobre un escenario ficticio debido a las limitaciones de red impuestas por la GVA. En casa se puede probar sobre dominios reales con:

```bash
whois ejemplo.com
nslookup ejemplo.com
```

## B antes y despues

# B1 Preparación

Torrent-Auditor (Ubuntu normal): 192.168.56.100

Torrent-Vulnerable (Metasploitable 2): 192.168.56.101

Se tomó una instantánea de Torrent-Vulnerable antes de modificarla (VirtualBox → Máquina → Tomar instantánea).

Se comprobó conectividad desde Torrent-Auditor:

```bash
ping 192.168.56.101
ping -c 3 192.168.56.101
```

#B2 medición antes
En torrent auditor:
```bash
sudo nmap -sV 192.168.56.101 -oN antes.txt
sudo namp -sV 192.168.56.101 > antes.txt
```

resultado:

|   Puerto | Servicio    | Versión                             |
| -------: | ----------- | ----------------------------------- |
|   21/tcp | ftp         | vsftpd 2.3.4                        |
|   22/tcp | ssh         | OpenSSH 4.7p1 Debian 8ubuntu1       |
|   23/tcp | telnet      | Linux telnetd                       |
|   25/tcp | smtp        | Postfix smtpd                       |
|   53/tcp | domain      | ISC BIND 9.4.2                      |
|   80/tcp | http        | Apache httpd 2.2.8 (Ubuntu) DAV/2   |
|  111/tcp | rpcbind     | 2 (RPC #100000)                     |
|  139/tcp | netbios-ssn | Samba smbd 3.X - 4.X                |
|  445/tcp | netbios-ssn | Samba smbd 3.X - 4.X                |
|  512/tcp | exec        | netkit-rsh rexecd                   |
|  513/tcp | login       | —                                   |
|  514/tcp | shell       | Netkit rshd                         |
| 1099/tcp | java-rmi    | GNU Classpath grmiregistry          |
| 1524/tcp | bindshell   | Metasploitable root shell           |
| 2049/tcp | nfs         | 2-4 (RPC #100003)                   |
| 2121/tcp | ftp         | ProFTPD 1.3.1                       |
| 3306/tcp | mysql       | MySQL 5.0.51a-3ubuntu5              |
| 5432/tcp | postgresql  | PostgreSQL DB 8.3.0 - 8.3.7         |
| 5900/tcp | vnc         | VNC (protocol 3.3)                  |
| 6000/tcp | X11         | (access denied)                     |
| 6667/tcp | irc         | UnrealIRCd                          |
| 8009/tcp | ajp13       | Apache Jserv (Protocol v1.3)        |
| 8180/tcp | http        | Apache Tomcat/Coyote JSP engine 1.1 |



#B3 contramedidas (en Torrent-Vulnerable,con sudo)
#Contramedida 1 — Inventariar y apagar servicios innecesarios

para listar lo que se esta escuchando aplicaremos el siguietne comando

```bash
sudo netstat -tulpn | grep -E "UDP|TCP"
```

esto nos dira los servicios en la maquina metasploitable el nombre del servicio junto con el puerto en donde esta para posteriormente pararlo y cerrarlo/matarlo

```bash
# Camino 1: preguntar al servicio
sudo nmap -sV -p 512 192.168.56.101

# Camino 2: mirar el portero (inetd/xinetd)
grep -v '^#' /etc/inetd.conf
grep -H disable /etc/xinetd.d/*
sudo grep -rn -w 512 /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d 2>/dev/null

# Camino 3: agenda del sistema
grep -w 512/tcp /etc/services

# Programa → paquete
dpkg -S /usr/sbin/tcpd
```


Resultado:

512/tcp = exec → /usr/sbin/tcpd /usr/sbin/in.rexecd

513/tcp = login → /usr/sbin/tcpd /usr/sbin/in.rlogind

514/tcp = shell → /usr/sbin/tcpd /usr/sbin/in.rshd

23/tcp = telnet → /usr/sbin/tcpd /usr/sbin/in.telnetd

dpkg -S /usr/sbin/tcpd → paquete tcpd

tabla de desicion:

| **Puerto** | **Servicio**         | **¿Necesario?** | **Decisión**                   |
| :--------- | :------------------- | :-------------- | :----------------------------- |
| 21         | vsftpd               | No              | Apagar                         |
| 22         | ssh                  | **Sí**          | Mantener                       |
| 23         | telnet (inetd)       | No              | Comentar en `/etc/inetd.conf`  |
| 25         | smtp                 | No              | Apagar                         |
| 53         | bind                 | No              | Apagar                         |
| 80         | apache               | **Sí**          | Mantener                       |
| 111        | rpcbind              | No              | Matar proceso                  |
| 139/445    | samba                | No              | Apagar                         |
| 512        | exec (inetd)         | No              | Comentar en `/etc/inetd.conf`  |
| 513        | login (inetd)        | No              | Comentar en `/etc/inetd.conf`  |
| 514        | shell (inetd)        | No              | Comentar en `/etc/inetd.conf`  |
| 1099       | java-rmi             | No              | Matar proceso                  |
| 1524       | bindshell (backdoor) | No              | Comentar en `/etc/inetd.conf`  |
| 2049       | nfs                  | No              | Apagar                         |
| 2121       | proftpd              | No              | Apagar                         |
| 3306       | mysql                | No              | Apagar                         |
| 5432       | postgresql           | No              | Apagar                         |
| 5900       | vnc                  | No              | Matar proceso                  |
| 6000       | X11                  | No              | Matar proceso (asociado a VNC) |
| 6667       | irc                  | No              | Matar proceso                  |
| 8009/8180  | tomcat               | No              | Apagar                         |


para apagar un servicio:
```bash
sudo /etc/init.d/samba stop
```
para matarlo y hacer que no regrese despue de un reinicio:
```bash
sudo update-rc.d -f samba remove
```

lo mismo para el resto a excepcion del que pone el tabla que es comentar en ciertos archivos para que no vuelvan a iniciarse

#5. Servicios "a demanda" (inetd) — telnet, rsh, rlogin, exec, tftp, ingreslock (backdoor):

```bash
sudo nano /etc/inetd.conf

# Comentar con # todas las líneas activas
sudo /etc/init.d/openbsd-inetd restart
```
6. Servicios sin script init.d (VNC, X11, java-rmi, rpcbind):
```bash
# Localizar procesos
ps aux | grep -i -E "vnc|X|rmi|rpc|java"

# Matar por PID o por nombre
sudo kill -9 <PID>
sudo pkill -9 Xtightvnc
```
estos serian algunos ejemplos, lo mismo aplicaria para los demas

para verificar que solo quedan los que queremos simplmente usar el mismo comando de antes para ver cuales estaban activos con netstat y grep


#Contramedida 2 — Bloquear el ping (obligatoria)

```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

y luego desde otra maquina hacemos ping para verificar de que ya quedo deshabilitado

#B4 medicion despues

```bash
sudo nmap -sV 192.168.56.101 -oN despues.txt
diff antes.txt despues.txt
```

| **Aspecto**                  | **Antes**                                                          | **Contramedida aplicada**                                                             | **Después**                               |
| :--------------------------- | :----------------------------------------------------------------- | :------------------------------------------------------------------------------------ | :---------------------------------------- |
| Puertos abiertos             | 23                                                                 | Apagado de servicios innecesarios (`init.d` + `update-rc.d` + `inetd.conf` + `pkill`) | 2 (22 y 80)                               |
| Servicios/versiones visibles | FTP, Telnet, Samba, MySQL, VNC, Tomcat, IRC, PostgreSQL, NFS, etc. | Ocultar banners + apagar servicios                                                    | Solo SSH (OpenSSH 4.7p1) y Apache (2.2.8) |
| Respuesta al ping            | Sí                                                                 | `iptables -A INPUT -p icmp --icmp-type echo-request -j DROP`                          | No (timeout, 100% packet loss)            |


##Reflexion
