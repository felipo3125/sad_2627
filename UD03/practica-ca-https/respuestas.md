# Respuestas — Práctica UD3

## Parte 1.3

1. **Diferencias:**
   - `ca.key` / `srv.key`: **secretas**.
   - `ca.crt` / `srv.crt`: **públicas**.

2. **Robo de `ca.key`:** emitir certificados falsos. Solución: revocar la CA entera, regenerar y redistribuir.

3. **Riesgo de descargar la CA por HTTP sin verificar huella:** MITM que sustituya el fichero. Mitigación: comparar huella SHA-256.

4. **CA y servidor en la misma máquina:** mala idea. La CA debe estar separada (idealmente offline).

## Parte 3

1. **¿Por qué el navegador confía en Let's Encrypt y no en Torrent S.L.?** Porque la raíz de Let's Encrypt viene preinstalada; la de Torrent S.L. hay que importarla.

2. **¿Por qué 90 días en Let's Encrypt?** Para limitar el impacto de una clave comprometida.

3. **Servidor sin Internet → CA propia. Web pública → Let's Encrypt.**

4. **Ataques que HTTPS no impide:** phishing y XSS.

5. **Empresa de 3 personas:** CA propia solo si tiene servicios internos (VPN, intranet). Para web pública, Let's Encrypt es más barato de mantener.
EOF
