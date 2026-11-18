# TryHackMe — Fools Mate | Writeup

> **Plataforma:** TryHackMe
> **Máquina:** Fools Mate
> **Sistema operativo:** Linux
> **Técnicas utilizadas:** Nmap, Gobuster, análisis de JavaScript del lado del cliente, consumo directo de API con curl
> **Estado:** vector identificado. La confirmación del último movimiento y la flag están pendientes.

---

## Índice

1. Reconocimiento
2. Enumeración web
3. Análisis del cliente
4. Interacción directa con la API
5. Flag

---

## 1. Reconocimiento

```bash
nmap -sC -sV <IP>
```

| Puerto | Servicio | Detalle |
|--------|----------|---------|
| 22 | SSH | OpenSSH 9.6p1 Ubuntu |
| 80 | HTTP | Node.js Express, título "Endgame Trainer" |

El nombre de la sala hace referencia al jaque mate del tonto, la secuencia de mate más corta del ajedrez, y la aplicación es un entrenador de finales con un tablero.

## 2. Enumeración web

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x js,json,txt
```

Solo aparecieron recursos estáticos: `/css/`, `/js/`, `/vendor/` e `index.html`. Resolver el tablero desde la interfaz dispara una ventana falsa de apagado del sistema. Es una broma de la sala y no afecta a la máquina.

## 3. Análisis del cliente

Al revisar `app.js` se vio que la función `preMoveCheck()` valida el mate solo en el navegador y nunca llama a `sendMove()`. Por lo tanto, la validación real y la flag dependen del endpoint `/api/move` del servidor, y la interfaz no sirve para resolver el reto.

## 4. Interacción directa con la API

El primer intento enviando una torre de a1 a a8 con `curl` devolvió `{"ok":false,"error":"illegal move"}` junto con la posición real del servidor en notación FEN:

```
6k1/5ppp/8/8/8/8/5PPP/R5K1 w - - 0 1
```

La posición del servidor no coincidía con la que se asumió a partir de la interfaz. Con las blancas a mover (torre en a1, rey en g1) y las negras con el rey en g8 y peones en f7, g7 y h7, la jugada ganadora es `Ra8#`: la fila trasera está libre y la torre cubre g8 y f8, sin casillas de escape para el rey negro.

```bash
curl -X POST http://<IP>/api/move \
  -H "Content-Type: application/json" \
  -d '{"from":"a1","to":"a8"}'
```

El cuerpo exacto del JSON debe copiarse del formato que acepte el endpoint, porque el trabajo previo no registró el esquema completo.

## 5. Flag

| Flag | Valor |
|------|-------|
| Flag de la sala | Pendiente de confirmar |

La respuesta del servidor a `Ra8#` no quedó registrada, por lo que este writeup debe completarse con esa salida.
