# TryHackMe — Silver Platter | Writeup

> **Plataforma:** TryHackMe
> **Máquina:** Silver Platter
> **Sistema operativo:** Linux (Ubuntu)
> **Técnicas utilizadas:** Nmap, enumeración manual, Silverpeas 6.3.1, bypass de autenticación, revisión de logs, reutilización de credenciales, sudo

---

## Índice

1. Reconocimiento
2. Enumeración web
3. Bypass de autenticación en Silverpeas
4. Credenciales en mensajes enviados y acceso SSH
5. Movimiento lateral hacia tyler
6. Escalada a root
7. Flags
8. Observaciones

---

## 1. Reconocimiento

```bash
nmap -sC -sV -oN initial_scan.txt <IP>
nmap -p- --min-rate=1000 -oN full_scan.txt <IP>
```

| Puerto | Servicio | Detalle |
|--------|----------|---------|
| 22 | SSH | OpenSSH 8.9p1 Ubuntu |
| 80 | HTTP | nginx 1.18.0, sitio "Hack Smarter Security" |
| 8080 | HTTP-proxy | Responde error a peticiones genéricas |

El escaneo completo confirmó que no había servicios ocultos en otros puertos.

## 2. Enumeración web

El sitio del puerto 80 usa la plantilla Dimension de HTML5 UP, por lo que los archivos README y LICENSE no aportaron nada. La información útil estaba en el texto de las secciones "About" y "Contact": el nombre del fundador (Tyler Ramsbey), la mención al CMS Silverpeas y un nombre de usuario (`scr1ptkiddy`). Un filtrado con `grep` había ocultado esa información, por lo que la lectura manual fue lo que desbloqueó la máquina.

Con eso se localizó `/silverpeas` en el puerto 8080, que corresponde a Silverpeas 6.3.1. La ruta `/console` es un señuelo que redirige a un 404 falso y no conviene perder tiempo ahí.

## 3. Bypass de autenticación en Silverpeas

Se trabajó únicamente con `curl`. Primero se establece la sesión y luego se envía el login omitiendo por completo el parámetro `Password` (no vacío, ausente):

```bash
curl -c cookies.txt -b cookies.txt http://<IP>:8080/silverpeas/ -o /dev/null

curl -c cookies.txt -b cookies.txt -i \
  -d "Login=SilverAdmin&DomainId=0" \
  http://<IP>:8080/silverpeas/AuthenticationServlet
```

La respuesta es un 302 hacia `MainFrame.jsp`, lo que confirma una sesión de administrador sin conocer la clave. La cookie `JSESSIONID` del archivo se reutilizó en el navegador.

## 4. Credenciales en mensajes enviados y acceso SSH

Dentro del panel, la sección de notificaciones enviadas contenía en texto plano las credenciales SSH del usuario `tim`. Con ellas se entró por SSH y se leyó `user.txt`. Al revisar `/etc/passwd` apareció también el usuario `tyler`.

## 5. Movimiento lateral hacia tyler

Los logs del sistema registraban comandos `docker run` con una variable `DB_PASSWORD` en claro. Ese valor resultó ser la contraseña de `tyler`:

```bash
su tyler
```

## 6. Escalada a root

```bash
sudo -l
```

`tyler` tiene permisos `ALL:ALL`, por lo que `sudo su` basta para obtener una shell de root y leer `root.txt`.

## 7. Flags

| Flag | Valor |
|------|-------|
| user.txt | `THM{********}` |
| root.txt | `THM{********}` |

## 8. Observaciones

- La máquina trae un `PATH` roto a propósito. Se restablece con `export PATH=$PATH:/usr/bin:/bin:/usr/sbin:/sbin`.
- Las URLs con doble barra (por ejemplo `Main//look/jsp/`) devolvían respuestas vacías. Usar barras simples o `-L`.
- `ffuf` no ignora las líneas de comentario de las wordlists, a diferencia de `gobuster`, y genera falsos positivos. Para confirmarlo se compara el código de respuesta de una ruta inexistente aleatoria contra la sospechosa.
- Las credenciales en texto plano y la reutilización de contraseñas fueron las fallas determinantes de esta cadena.
