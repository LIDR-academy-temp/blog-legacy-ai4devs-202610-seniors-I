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

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Lee lo que hay que hacer en este ejercicio y prepara un plannotator para ir punto por punto y guiarme en como hacerlo
```

**Qué salió:** leyó README, prompts.md, CLAUDE.md y el comando /refactor, comprobó el entorno (fork, deps, base, gh) y escribió una guía de 8 pasos en un markdown, pero Plannotator no arrancó por un bloqueo de permisos (`$.process.run`), así que la guía quedó en un archivo local y ofreció seguir por el chat.

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
A partir de ahora, anade los prompts y las respuestas en prompts.md utilizando el mismo formato que el ejemplo
```

**Qué salió:** borró el ejemplo y registró en este archivo los prompts 1 y 2 con un resumen de la respuesta.

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
dame un ejemplo de un hallazgo siguiento el formato que indica el readme
```

**Qué salió:** leyó las rutas de comentarios, la migración y Comentarios.jsx, y devolvió un hallazgo de ejemplo con las 6 casillas: el POST relee el comentario con `order by id desc limit 1` (routes.ts:330) en vez de por su id.

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
acorde con el ejercicio, encuentra los hallazgos en el codigo de comentarios e indicame donde estan
```

**Qué salió:** leyó routes.ts entero, la migración, la semilla, el handler, el test existente y los componentes del front, y devolvió 18 hallazgos priorizados con archivo y línea, más correcciones a tres de los míos (respuestas que sí se ven, cuerpo vacío que sí se rechaza).
