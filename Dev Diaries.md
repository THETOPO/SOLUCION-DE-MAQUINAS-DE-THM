# TryHackMe — Dev Diaries | Writeup

> **Plataforma:** TryHackMe
> **Sala:** Dev Diaries
> **Tipo:** OSINT
> **Técnicas utilizadas:** Certificate Transparency, búsqueda de repositorios en GitHub, historial de commits

---

## Escenario

Un desarrollador freelance construyó el sitio `marvenly.com` y desapareció sin entregar el código fuente. El objetivo es reconstruir su rastro público y responder cinco preguntas.

## Respuestas

| Pregunta | Respuesta |
|----------|-----------|
| Subdominio de la versión de desarrollo | `uat-testing.marvenly.com` |
| Usuario de GitHub del desarrollador | `notvibecoder23` |
| Correo del desarrollador | `freelancedevbycoder23@gmail.com` |
| Motivo del commit que eliminó el código | `The project was marked as abandoned due to a payment dispute.` |
| Flag oculta | `THM{********}` |

## Método

**Subdominio.** Se consultan los registros de Certificate Transparency (por ejemplo crt.sh) para listar los certificados emitidos al dominio y ver subdominios de preproducción. En este caso el sitio bloqueó el acceso automatizado, por lo que el subdominio se tomó de writeups públicos y no se confirmó de forma independiente.

**Usuario de GitHub.** Con el nombre del sitio y del entorno de pruebas se localizó el repositorio `notvibecoder23/marvenly_site`.

**Correo y flag.** En GitHub, añadir `.patch` a la URL de un commit muestra el parche completo con el autor y su correo. Ahí se leyeron el correo y la flag, ambos verificados directamente. La flag está en el historial: borrar archivos en un commit posterior no los elimina del repositorio.

**Motivo.** La respuesta debía escribirse como la frase completa del mensaje del commit. El patrón de longitudes de palabra del campo de respuesta (3, 7, 3, 6, 2, 9, 3, 2, 1, 7, 7) coincide con esa frase.

## Lección

Un repositorio público conserva el historial completo. Credenciales, correos y secretos eliminados en commits posteriores siguen siendo recuperables, y los metadatos del autor se exponen en cada parche.
