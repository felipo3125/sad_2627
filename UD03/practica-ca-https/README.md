# Práctica UD3 — Certificados y HTTPS · Torrent S.L.

**Módulo:** Seguridad y Alta Disponibilidad · 2º ASIX  
**Entorno:** Ubuntu 24.04 LTS (VM VirtualBox)  
---

## 1. Contexto y objetivo

El servidor `srv-hmi` de Torrent S.L. publica una página interna (HMI) con el estado de la planta. Originalmente se servía por HTTP en texto plano, lo que permitía a cualquiera en la red leer credenciales y modificar respuestas.

El objetivo de esta práctica es:

1. Montar una **autoridad de certificación (CA) propia** para Torrent S.L.
2. Emitir y firmar un **certificado de servidor** para `hmi.torrent.local`.
3. Configurar **Apache con HTTPS** usando ese certificado.
4. Verificar el funcionamiento con `curl`, `openssl` y un navegador (Firefox).

---

## 2. Entorno

| Elemento | Valor |
|---|---|
| Sistema | Ubuntu 24.04 LTS (Noble) |
| Apache | 2.4.58 (Ubuntu) |
| OpenSSL | 3.0.13 |
| IP del servidor (VM) | `10.0.2.15` |
| Nombre del servicio | `hmi.torrent.local` |
| DocumentRoot | `/var/www/html/` |

---

## 3. Estructura de la PKI propia

```
~/ca/
├── certs/
│   ├── ca.crt        (público)    certificado raíz de la CA
│   └── srv.crt       (público)    certificado del servidor firmado por la CA
├── private/
│   ├── ca.key        (SECRETO)    clave privada de la CA (cifrada con passphrase)
│   └── srv.key       (SECRETO)    clave privada del servidor
├── srv.csr           (público)    petición de certificado
└── srv.ext           (público)    extensiones SAN del certificado
```

- **Secretos:** `ca.key` y `srv.key` nunca salen de la máquina ni se suben al repositorio.
- **Públicos:** `ca.crt` y `srv.crt` se pueden distribuir sin riesgo.

---

## 4. Pasos ejecutados

### 4.1 Crear la carpeta de la CA

```bash
mkdir -p ~/ca/private ~/ca/certs
chmod 700 ~/ca/private
```

La subcarpeta `private` tiene permisos `700` para que solo el usuario pueda entrar en ella. Aun así, `ca.key` va cifrada con `-aes256` como **defensa en profundidad**.

### 4.2 Generar la clave de la CA (cifrada)

```bash
openssl genrsa -aes256 -out private/ca.key 4096
```

- RSA de **4096 bits** (la CA vive muchos años).
- `-aes256` cifra el fichero con una passphrase. **Si se pierde, no se puede recuperar.**

### 4.3 Crear el certificado raíz autofirmado (10 años)

```bash
openssl req -x509 -new -key private/ca.key -sha256 -days 3650 \
  -out certs/ca.crt \
  -subj "/C=ES/O=Torrent S.L./CN=Torrent S.L. CA Raiz" \
  -addext "basicConstraints=critical,CA:TRUE" \
  -addext "keyUsage=critical,keyCertSign,cRLSign"
```

- `Subject` = `Issuer` (autofirmado, normal en una raíz).
- `CA:TRUE` y `keyCertSign` permiten que esta clave firme otros certificados.
- Validez: **10 años** (`notAfter = Nov 2036`).

### 4.4 Clave del servidor y CSR

```bash
openssl genrsa -out private/srv.key 2048
openssl req -new -key private/srv.key -out srv.csr \
  -subj "/C=ES/O=Torrent S.L./CN=hmi.torrent.local"
```

- `srv.key` **sin passphrase** (Apache debe arrancar solo).
- `srv.csr` contiene la clave pública y la identidad, **nunca la privada**.

### 4.5 Fichero de extensiones SAN

```bash
cat > srv.ext <<'EOF'
subjectAltName = DNS:hmi.torrent.local, IP:10.0.2.15
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
EOF
```

Los navegadores modernos ignoran el `CN` y validan el **SAN** (Subject Alternative Name). Por eso incluimos tanto el nombre como la IP.

### 4.6 La CA firma el certificado del servidor

```bash
openssl x509 -req -in srv.csr \
  -CA certs/ca.crt -CAkey private/ca.key -CAcreateserial \
  -out certs/srv.crt -days 365 -sha256 -extfile srv.ext
```

Verificación:

```bash
openssl verify -CAfile certs/ca.crt certs/srv.crt
# → certs/srv.crt: OK
```

### 4.7 Instalación en Apache

```bash
sudo cp ~/ca/certs/srv.crt /etc/ssl/certs/torrent-srv.crt
sudo cp ~/ca/private/srv.key /etc/ssl/private/torrent-srv.key
sudo chmod 600 /etc/ssl/private/torrent-srv.key
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo nano /etc/apache2/sites-available/default-ssl.conf
# → SSLCertificateFile      /etc/ssl/certs/torrent-srv.crt
# → SSLCertificateKeyFile   /etc/ssl/private/torrent-srv.key
sudo apache2ctl configtest
sudo systemctl restart apache2
```

Apache queda escuchando en `*:80` y `*:443`.

---

## 5. Verificaciones

### 5.1 `curl` — el cliente debe rechazar la CA desconocida

```bash
$ touch /tmp/vacio.pem
$ curl -I --cacert /tmp/vacio.pem https://10.0.2.15
curl: (77) error setting certificate file: /tmp/vacio.pem
```

> Nota: `(77)` es porque `/dev/null`/fichero vacío no es un bundle PEM válido; el error equivalente del boletín sería `(60) SSL certificate problem: unable to get local issuer certificate`. Con `openssl s_client` se reproduce el `(60)` real (ver 5.3).

### 5.2 `curl` con la CA correcta

```bash
$ curl -I --cacert ~/ca/certs/ca.crt https://10.0.2.15
HTTP/1.1 200 OK
...

$ curl -I --cacert ~/ca/certs/ca.crt \
    --resolve hmi.torrent.local:443:10.0.2.15 https://hmi.torrent.local
HTTP/1.1 200 OK
```

### 5.3 `openssl s_client`

**Sin la CA (lo que ve un atacante):**

```bash
$ openssl s_client -connect 10.0.2.15:443 -servername hmi.torrent.local </dev/null 2>/dev/null | grep 'Verify return code'
Verify return code: 21 (unable to verify the first certificate)
```

**Con la CA:**

```bash
$ openssl s_client -connect 10.0.2.15:443 -servername hmi.torrent.local \
    -CAfile ~/ca/certs/ca.crt </dev/null 2>/dev/null \
    | grep -E ' s:| i:|Verify return code|Cipher|Protocol'
 s:C = ES, O = Torrent S.L., CN = hmi.torrent.local
 i:C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Verify return code: 0 (ok)
```

- TLS **1.3**
- Cifrado **TLS_AES_256_GCM_SHA384**
- **Verify return code: 0 (ok)**

### 5.4 Navegador (Firefox)

| Estado | Evidencia |
|---|---|
| Warning inicial (`SEC_ERROR_UNKNOWN_ISSUER`) | `evidencias/firefox-warning.png` |
| Candado tras importar la CA | `evidencias/firefox-candado.png` |
| Detalles del certificado | `evidencias/firefox-certificado.png` |

- **Emisor:** `Torrent S.L. CA Raiz`
- **Sujeto:** `hmi.torrent.local`
- **SAN:** `hmi.torrent.local`, `10.0.2.15`
- **Huella SHA-256 de la CA:**
  `7A:9E:99:9B:20:2F:F8:4D:AD:D5:0E:D7:C6:C1:B2:8A:21:ED:6C:72:CF:F7:BC:AA:86:C9:57:95:41:2C:5D:0E`

> **Nota sobre el enlace `630580e6.0`:** al copiar `srv.crt` a `/etc/ssl/certs/`, el sistema crea automáticamente un enlace simbólico `630580e6.0 -> torrent-srv.crt` porque ese directorio está indexado por `openssl rehash`. Como consecuencia, `curl` **también** confía en nuestra CA sin `--cacert`. Por eso el error `(60)` del boletín se reproduce con `openssl s_client` (5.3) o con `--cacert /tmp/vacio.pem` (5.1).

---

## 6. Respuestas a las preguntas del boletín

### 6.1 Parte 1.3

1. **Diferencia entre `ca.key`, `ca.crt`, `srv.key` y `srv.crt`:**
   - `ca.key` y `srv.key` son **secretas**. Si se filtran, cualquiera puede firmar certificados en nombre de Torrent S.L. (la de la CA) o suplantar al servidor (la del servidor).
   - `ca.crt` y `srv.crt` son **públicas**. Se reparten sin riesgo.

2. **Si roban `ca.key` (sin passphrase):** el atacante puede emitir certificados válidos para cualquier dominio en nombre de Torrent S.L. La empresa tendría que **revocar toda la cadena**, regenerar la CA, redistribuir el nuevo `ca.crt` y revocar los certificados emitidos.

3. **Descargar `ca.crt` por HTTP sin verificar la huella:** un atacante en la red podría sustituir el fichero por su propia CA y, si el usuario la importa, el navegador confiaría en él. Mitigación: **comparar la huella SHA-256** por un canal distinto (como hicimos).

4. **¿Es buena idea que CA y servidor web compartan máquina?** No. Si el servidor se compromete, también se lleva `ca.key`. En producción la CA debe estar **separada y, a ser posible, offline**.

### 6.2 Parte 3 (finales)

1. **¿Por qué el navegador confía en Let's Encrypt y no en la CA de Torrent S.L.?** Porque la raíz de Let's Encrypt viene **preinstalada** en los sistemas/navegadores. La de Torrent S.L. hay que **importarla manualmente**.

2. **¿Por qué los certificados de Let's Encrypt duran 90 días?** Para **limitar el daño** si la clave privada se ve comprometida: caduca pronto y se renueva automáticamente.

3. **Servidor sin Internet:** usar **CA propia**. Para la web pública de un cliente: **Let's Encrypt** (o una CA pública).

4. **Dos ataques que HTTPS no impide:** phishing (el sitio puede ser legítimo pero falso) y XSS en la propia aplicación. HTTPS protege el canal, no el contenido ni al usuario.

5. **¿Una empresa de 3 personas montaría una CA propia?** Solo si tiene servicios internos (VPN, intranet) que no necesitan ser públicos. En la Fase 2 usaremos la CA para la VPN.

---

## 7. Limpieza realizada

- [x] CA `Torrent S.L.` **eliminada de Firefox** al terminar.
- [x] `ca.key` y `srv.key` **no subidos** al repositorio.
- [x] Backup de la CA en `~/ca-torrent-backup.tar.gz` (no versionado).

---

## 8. Repositorio

- **Contenido:** `README.md`, `.gitignore`, `evidencias/`, `respuestas.md`
- **No versionado:** `*.key`, `*.csr`, `*.crt`, `ca-torrent-backup.tar.gz`, `.acme.sh/`

---

## 9. Conclusiones

- La CA propia permite HTTPS **interno** para la planta sin depender de Internet.
- El **SAN** es obligatorio en navegadores modernos.
- `curl` y `openssl` son herramientas clave para diagnosticar TLS.
- HTTPS garantiza confidencialidad e integridad del canal, **no** la autenticidad del contenido ni la seguridad del usuario final.
