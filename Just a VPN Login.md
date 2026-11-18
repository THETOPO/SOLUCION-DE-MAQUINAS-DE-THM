# TryHackMe — Just a VPN Login | Writeup

> **Plataforma:** TryHackMe
> **Sala:** Just a VPN Login
> **Tipo:** SOC / respuesta a incidentes
> **Herramientas:** plataforma de inteligencia de amenazas TryDetectThis, MITRE ATT&CK, YARA

---

## Escenario

Llega una alerta por un inicio de sesión VPN inusual atribuido a Susan, del área de Marketing, mientras viajaba en Singapur. Hay que investigar la IP sospechosa y el hash de un binario asociado.

Hash del binario:

```
b8e02f2bc0ffb42e8cf28e37a26d8d825f639079bf6d948f8debab6440ee5630
```

## Hallazgos

| Elemento | Resultado |
|----------|-----------|
| ASN de la IP sospechosa | 212238, Datacamp Limited |
| Nombre del archivo malicioso | `zY9sqWs.exe` |
| Detección de Microsoft | `Trojan:Win32/LummaStealer` |
| Dominios ligados por el certificado HTTPS en el clúster de infraestructura | 151 |
| Informe relacionado | "Behind the Curtain: How Lumma Affiliates Operate" (Recorded Future) |
| Grupo con el que el autor empezó a colaborar en 2024 | GhostSocks |
| Infostealer/RAT para Android usado por un afiliado de México | CraxsRAT |
| Sub-técnica MITRE ATT&CK alineada con el servicio AnonRDP | T1583.003 |

## Veredicto

Verdadero positivo. El acceso fue posible por compromiso de credenciales: la usuaria instaló una falsa herramienta de "verificación de seguridad" en una red Wi-Fi pública, que resultó ser el infostealer Lumma.

## Respuestas por validar

Las preguntas 2, 4 y 10 de la sala se respondieron en parte por inferencia o con ayuda de un walkthrough público, y no con consulta directa a la plataforma. Antes de dar este writeup por definitivo hay que contrastarlas con el formato exacto que acepta TryHackMe, en particular la línea de condición de la regla YARA del autor `kevoreilly`.
