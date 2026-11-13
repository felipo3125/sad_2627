## Bloque A — Verificaciones con `curl` y `openssl`

### A.1 Entorno
```
OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024)
Server version: Apache/2.4.58 (Ubuntu)
```

### A.2 `curl` sin la CA (debe fallar)
```bash
$ touch /tmp/vacio.pem
$ curl -I --cacert /tmp/vacio.pem https://10.0.2.15
curl: (77) error setting certificate file: /tmp/vacio.pem
```
> Nota: el código 77 se debe a que el fichero `/tmp/vacio.pem` no contiene un bundle PEM válido. El error equivalente del boletín `(60) unable to get local issuer certificate` se reproduce con `openssl s_client` (ver A.5).

### A.3 `curl` con la CA correcta (debe dar 200)
```bash
$ curl -I --cacert ~/ca/certs/ca.crt https://10.0.2.15
HTTP/1.1 200 OK
Date: Fri, 13 Nov 2026 15:24:45 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Thu, 12 Nov 2026 18:33:52 GMT
ETag: "1b6e-660188dae7630"
Accept-Ranges: bytes
Content-Length: 7022
Vary: Accept-Encoding
Content-Type: text/html
```

### A.4 `curl` por nombre (con `--resolve`)
```bash
$ curl -I --cacert ~/ca/certs/ca.crt \
    --resolve hmi.torrent.local:443:10.0.2.15 https://hmi.torrent.local
HTTP/1.1 200 OK
Date: Fri, 13 Nov 2026 15:26:08 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Thu, 12 Nov 2026 18:33:52 GMT
ETag: "1b6e-660188dae7630"
Accept-Ranges: bytes
Content-Length: 7022
Vary: Accept-Encoding
Content-Type: text/html
```

### A.5 `openssl s_client` sin la CA (lo que ve un atacante)
```bash
$ openssl s_client -connect 10.0.2.15:443 -servername hmi.torrent.local </dev/null 2>/dev/null | grep 'Verify return code'
Verify return code: 21 (unable to verify the first certificate)
    Verify return code: 21 (unable to verify the first certificate)
```

### A.6 `openssl s_client` con la CA (handshake completo)
```bash
$ openssl s_client -connect 10.0.2.15:443 -servername hmi.torrent.local \
    -CAfile ~/ca/certs/ca.crt </dev/null 2>/dev/null \
    | grep -E ' s:| i:|Verify return code|Cipher|Protocol'
 0 s:C = ES, O = Torrent S.L., CN = hmi.torrent.local
   i:C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Verify return code: 0 (ok)
```

**Conclusión del Bloque A:**
- El cliente rechaza la conexión cuando no conoce la CA (`Verify return code: 21`).
- El cliente acepta la conexión cuando se le proporciona la CA correcta (`Verify return code: 0`).
- El protocolo negociado es **TLS 1.3** con cifrado **TLS_AES_256_GCM_SHA384**, lo que confirma que el servidor Apache está sirviendo HTTPS de forma segura.


imagenes de evidencia
<img width="955" height="915" alt="firefox advertencia" src="https://github.com/user-attachments/assets/d98e5b47-b028-48f3-94b4-99d77fc878b1" />
<img width="955" height="915" alt="importar certifiado a firefox para la practica" src="https://github.com/user-attachments/assets/9a173801-80dc-471c-a6b0-593276d5c6b3" />
<img width="955" height="915" alt="aceptar en confiar en la autoridad certificadora" src="https://github.com/user-attachments/assets/75c7208e-9646-41e4-8311-22cea9a48286" />
<img width="955" height="915" alt="despues del certificado para conexion segura y ver la pagina" src="https://github.com/user-attachments/assets/98eaf3a6-c55c-444d-ae75-53cf5b715524" />
<img width="955" height="915" alt="detalles del certificado con la pagina" src="https://github.com/user-attachments/assets/edb493f7-ea2f-4f8a-ab76-5e5928808fb0" />
