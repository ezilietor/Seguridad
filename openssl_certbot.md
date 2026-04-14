# OpenSSL & Certbot — Cheatsheet

---

## OpenSSL

### Claves & Certificados

```bash
# Generar clave privada RSA
openssl genrsa -out clave.key 2048

# Generar CSR (solicitud de certificado)
openssl req -new -key clave.key -out solicitud.csr

# Generar certificado autofirmado
openssl req -x509 -new -key clave.key -out cert.crt -days 365
```

### Inspeccionar

```bash
# Ver info de un certificado
openssl x509 -in cert.crt -text -noout

# Ver fechas de expiración
openssl x509 -in cert.crt -noout -dates

# Ver info de un CSR
openssl req -in solicitud.csr -text -noout

# Ver certificado de un servidor remoto
openssl s_client -connect dominio.com:443
```

### Cifrado

```bash
# Cifrar archivo
openssl enc -aes-256-cbc -in archivo.txt -out archivo.enc

# Descifrar archivo
openssl enc -d -aes-256-cbc -in archivo.enc -out archivo.txt

# Generar hash SHA-256
openssl dgst -sha256 archivo.txt

# Firmar archivo
openssl dgst -sha256 -sign clave.key -out firma.sig archivo.txt

# Verificar firma
openssl dgst -sha256 -verify clave_publica.pem -signature firma.sig archivo.txt
```

### Convertir formatos

```bash
# PFX/P12 a PEM
openssl pkcs12 -in cert.pfx -out cert.pem -nodes

# PEM a DER
openssl x509 -in cert.pem -outform DER -out cert.der
```

---

## Certbot

### Instalación

```bash
# Ubuntu/Debian
sudo apt install certbot

# Con plugin Nginx
sudo apt install certbot python3-certbot-nginx

# Con plugin Apache
sudo apt install certbot python3-certbot-apache
```

### Obtener certificado

```bash
# Con Nginx
sudo certbot --nginx -d dominio.com -d www.dominio.com

# Con Apache
sudo certbot --apache -d dominio.com -d www.dominio.com

# Standalone (sin servidor web)
sudo certbot certonly --standalone -d dominio.com

# Webroot (servidor ya corriendo)
sudo certbot certonly --webroot -w /var/www/html -d dominio.com

# Manual
sudo certbot certonly --manual -d dominio.com
```

### Gestionar

```bash
# Listar certificados
sudo certbot certificates

# Renovar todos
sudo certbot renew

# Simulación sin cambios reales
sudo certbot renew --dry-run

# Forzar renovación
sudo certbot renew --force-renewal

# Renovar uno específico
sudo certbot renew --cert-name dominio.com

# Revocar certificado
sudo certbot revoke --cert-path /etc/letsencrypt/live/dominio.com/cert.pem

# Eliminar certificado
sudo certbot delete --cert-name dominio.com
```

### Cert de Let's Encrypt con OpenSSL

```bash
# Ver info del certificado
openssl x509 -in /etc/letsencrypt/live/dominio.com/cert.pem -text -noout

# Ver fechas de expiración
openssl x509 -in /etc/letsencrypt/live/dominio.com/cert.pem -noout -dates
```

### Rutas importantes

| Archivo | Ruta |
|---|---|
| Certificado | `/etc/letsencrypt/live/dominio.com/cert.pem` |
| Cadena completa | `/etc/letsencrypt/live/dominio.com/fullchain.pem` |
| Clave privada | `/etc/letsencrypt/live/dominio.com/privkey.pem` |
| Configuración | `/etc/letsencrypt/renewal/dominio.com.conf` |
| Logs | `/var/log/letsencrypt/letsencrypt.log` |

### Renovación automática

```bash
# Ver timer systemd
sudo systemctl status certbot.timer

# Agregar al cron manualmente
0 3 * * * certbot renew --quiet
```

### Flags útiles

| Flag | Descripción |
|---|---|
| `--agree-tos` | Acepta términos automáticamente |
| `--email tu@email.com` | Correo para notificaciones |
| `--non-interactive` | Sin prompts (para scripts) |
| `--quiet` | Sin salida en consola |
| `--staging` | Servidor de pruebas (no consume límite) |
