# TryHackMe — Billing | Writeup

> **Plataforma:** TryHackMe
> **Máquina:** Billing
> **Sistema operativo:** Linux
> **Técnicas utilizadas:** Nmap, whatweb, searchsploit, CVE-2023-30258 (inyección de comandos en MagnusBilling), reverse shell, sudo sobre fail2ban-client, SUID
> **Estado:** user.txt obtenida. La escalada a root quedó documentada hasta el disparo del ban y está pendiente de verificación final.

---

## Índice

1. Reconocimiento
2. Identificación de la aplicación
3. Explotación
4. Post-explotación
5. Escalada de privilegios
6. Flags

---

## 1. Reconocimiento

```bash
nmap -p- --min-rate=1000 <IP>
nmap -sC -sV -p 22,80,3306,5038 <IP>
```

| Puerto | Servicio | Comentario |
|--------|----------|------------|
| 22 | SSH | Sin credenciales en este punto |
| 80 | HTTP | Aplicación web |
| 3306 | MySQL | Expuesto, sin acceso directo |
| 5038 | Asterisk Manager Interface | Indica infraestructura VoIP |

## 2. Identificación de la aplicación

```bash
whatweb http://<IP>
```

La aplicación es MagnusBilling. Se buscó en la base de exploits locales:

```bash
searchsploit magnusbilling
```

Resultó afectada la versión 7.3.0 por la inyección de comandos CVE-2023-30258, que se explota sin autenticación.

## 3. Explotación

Se adaptó el exploit público leyendo primero cómo funciona la inyección. Las primeras pruebas de RCE ciego dieron resultados inconsistentes, porque el script vulnerable concatena contenido después del comando inyectado y lo rompe. La solución fue terminar la inyección con un `#` codificado en URL (`%23`), que comenta el resto de la línea. Con eso se verificó la ejecución mediante una comprobación fuera de banda y se lanzó una reverse shell hacia un listener de netcat, obteniendo acceso como el usuario `asterisk`.

## 4. Post-explotación

La enumeración local mostró dos hallazgos: el usuario `magnus` tiene el archivo `user.txt` en su directorio, y `asterisk` puede ejecutar `fail2ban-client` con sudo sin contraseña.

```bash
sudo -l
cat /home/magnus/user.txt
```

## 5. Escalada de privilegios

`fail2ban-client` con sudo permite modificar la configuración en ejecución y definir una acción que fail2ban corre como root cuando banea una IP. El procedimiento consiste en crear una acción personalizada cuyo comando de ban asigne el bit SUID a `/bin/bash`, asociarla a una jail y provocar un ban para que se dispare. Los comandos de creación de la acción y el ban se ejecutaron sin errores.

El paso siguiente es comprobar el SUID y abrir una shell privilegiada:

```bash
ls -l /bin/bash
/bin/bash -p
cat /root/root.txt
```

Esta última verificación no quedó registrada en el trabajo previo, por lo que debe confirmarse y completarse con la salida real antes de dar la máquina por cerrada.

## 6. Flags

| Flag | Valor |
|------|-------|
| user.txt | `THM{********}` |
| root.txt | Pendiente de confirmar |
