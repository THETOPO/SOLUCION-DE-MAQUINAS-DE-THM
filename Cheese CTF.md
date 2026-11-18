# TryHackMe — Cheese CTF | Writeup

> **Plataforma:** TryHackMe
> **Máquina:** Cheese CTF
> **Sistema operativo:** Linux
> **Técnicas utilizadas:** Nmap, Gobuster, LFI, filtros php://filter, SQL Injection, RCE con PHP filter chain, SSH authorized_keys, systemd timer, SUID, GTFOBins

---

## Índice

1. Reconocimiento
2. Enumeración web
3. Local File Inclusion
4. Bypass del login
5. De LFI a ejecución remota de comandos
6. Acceso como comte
7. Escalada de privilegios
8. Flags
9. Errores cometidos y lecciones

---

## 1. Reconocimiento

```bash
nmap -sC -sV -p- <IP>
```

El escaneo reportó los 65535 puertos como abiertos. Esto no es real: la máquina responde a todas las conexiones para estorbar el reconocimiento. Se verificó manualmente con `curl` y `nc` qué puertos atendían de verdad, y solo respondieron el 80 (Apache) y el 22 (SSH).

## 2. Enumeración web

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,html
```

Se encontraron `messages.html`, `orders.html`, `users.html` y `secret-script.php`. Este último recibía un parámetro `file`, lo que apuntaba a una inclusión de archivos.

## 3. Local File Inclusion

El script ejecutaba un `include($_GET['file'])` sin sanitizar. Con el wrapper `php://filter` se leyó el código fuente de `login.php` codificado en base64:

```
http://<IP>/secret-script.php?file=php://filter/convert.base64-encode/resource=login.php
```

## 4. Bypass del login

En el código de `login.php` el único filtro contra inyección SQL bloqueaba la palabra `OR`. Se evitó con un comentario de cierre en el campo de usuario:

```
admin'-- -
```

## 5. De LFI a ejecución remota de comandos

Se usó la herramienta `php_filter_chain_generator` de Synacktiv para construir una cadena de filtros que genera código PHP arbitrario a partir del propio parámetro `file`, convirtiendo el LFI en RCE. Con ese acceso se lanzó una reverse shell hacia un listener de netcat y se obtuvo una sesión como `www-data`.

## 6. Acceso como comte

En la enumeración local se encontró que el archivo `authorized_keys` del usuario `comte` tenía permisos de escritura para todos. Se generó un par de llaves en la máquina atacante, se agregó la llave pública a ese archivo y se entró por SSH como `comte`. Ahí se leyó `user.txt`.

## 7. Escalada de privilegios

`sudo -l` mostró que `comte` podía operar sin contraseña sobre un temporizador de systemd (`exploit.timer`) y sus unidades asociadas. La unidad estaba mal configurada, por lo que se corrigió para que ejecutara un comando propio como root. Al dispararse el temporizador se creó una copia de `xxd` con el bit SUID asignado a root. Con esa copia se pueden leer archivos arbitrarios como root siguiendo la técnica de GTFOBins, y así se obtuvo `root.txt`.

## 8. Flags

| Flag | Valor |
|------|-------|
| user.txt | `THM{********}` |
| root.txt | `THM{********}` |

## 9. Errores cometidos y lecciones

- Se confundió en varias ocasiones la terminal local de Kali con la de la reverse shell al ejecutar comandos. Conviene etiquetar o separar las ventanas.
- Se asumió un bloqueo sin revisar antes los permisos del archivo. Verificar con `ls -l` ahorra tiempo.
- Se intentó reutilizar una contraseña por SSH sin confirmar si la autenticación por contraseña estaba habilitada. Se debe revisar primero `sshd_config` o la respuesta del servicio.
