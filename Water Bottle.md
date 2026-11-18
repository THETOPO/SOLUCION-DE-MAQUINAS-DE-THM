# TryHackMe — Water Bottle | Writeup

> **Plataforma:** TryHackMe
> **Sala:** Water Bottle
> **Tipo:** OSINT geográfico

---

## Enunciado

Identificar una estación de recarga de agua cercana a Boni Avenue que el participante usó hasta 2014, con un número de contacto de doce dígitos que empieza con `63922`. La estación original fue reemplazada después por otro negocio en el mismo lugar. La flag combina el nombre de la estación y el número.

## Método

1. Búsqueda de estaciones de recarga en Boni Avenue (Mandaluyong) combinando el lugar con el prefijo `63922`.
2. Comparación de los candidatos con writeups públicos de la misma sala.
3. Verificación del número en la página de contacto del negocio identificado.

## Resultado

La estación corresponde a la sucursal de Aquabest en Boni, Mandaluyong. El formato de la flag es `THM{nombre_numero}`.

| Flag | Valor |
|------|-------|
| Flag de la sala | `THM{********}` |

## Limitaciones

El dato de 2014 no pudo verificarse de forma independiente. Además, las fuentes consultadas no coinciden en qué negocio reemplazó a la estación original. La identificación se apoya en la coincidencia del prefijo telefónico, la ubicación y los writeups públicos.
