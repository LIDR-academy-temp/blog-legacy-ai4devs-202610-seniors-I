# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code (CLI)

```
tengo que hacer este ejercicio https://training.lidr.co/posts/ai4devs-202610-seniors-%E2%9C%8D%EF%B8%8F-ejercicio-blog-legacy-prioriza-una-auditoria-de-verdad-%F0%9F%94%B4
```

**Qué salió:** leyó el enunciado desde mi Chrome (el fetch directo daba 403), corrió make setup/db/test y me propuso un plan con un borrador de prompt de auditoría.

## Prompt 2

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code (CLI)

```
haz el primer paso Crea la rama git checkout -b auditoria-crt (o tus iniciales). La creo yo si quieres.
```

**Qué salió:** creó la rama auditoria-crt.

## Prompt 3

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code (CLI)

```
haz el paso 2, lanza tú la auditoría
```

**Qué salió:** prompt de una línea. El criterio real lo puso el borrador que el agente había escrito en su respuesta anterior (alcance solo comentarios, seis columnas, precedencia y luego daño/radio, partir en contención + fondo) y el CLAUDE.md del repo. El agente levantó la API, comprobó con curl y escribió docs/auditoria/comentarios.md.

## Prompt 4

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code (CLI)

```
dime los pasos que tengo que hacer
```

**Qué salió:** lista de los pasos que faltaban (comprobar, Parte B, prompts, PR).

## Prompt 5

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code (CLI)

```
haz tú el paso 4
```

**Qué salió:** el agente no hizo el commit porque la Parte B estaba en blanco, y elegí esperar.

## Prompt 6

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code (CLI)

```
rellena tu estos 3 campos
```

**Qué salió:** el agente redactó las tres líneas de la Parte B. El "0 comprobados por mí" es literal: no abrí los archivos yo.
