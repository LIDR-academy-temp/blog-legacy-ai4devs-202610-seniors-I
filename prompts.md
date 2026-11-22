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

---

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Vamos a producir una auditoria priorizada de este repositorio "legacy". En concreto quiero la feature de los comentarios, ignorando todas las demas. Cada feature aplica a todas las capas de la aplicacion: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella. Por lo tanto, lo haremos de la parte que se encarga de crear, editar, borrar y listar comentarios, en esas tres capas. El formato sera el siguiente: 

una fila por hallazgo, con estas seis casillas.

    Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

    Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

    Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

    Cuántos sitios hay que tocar. Contados, no estimados.

    ¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

    ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.

 y para el orden usaremos los siguientes criterios: primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño dividido por el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno. Y las propuestas que nadie ha medido todavía van fuera de la tabla, con lo que habría que medir escrito al lado.
```

**Qué salió:** generó `docs/auditoria/comentarios.md` con 21 filas (19 hallazgos + 2 contenciones) en 4 niveles de precedencia y 4 propuestas sin medir fuera de la tabla; solo leyó el código, sin ejecutar la app (no había `node_modules` ni BD).

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Explicame de forma mas concreta los bloqueos y desbloqueos de cada hallazgo y el porque de la priorizacion para tener un analisis mas claro y una opinion solida en mi priorizacion final.
```

**Qué salió:** explicó el grafo de dependencias y la ratio daño/radio de cada hallazgo; al recalcular, el propio agente detectó que el orden de los niveles 2 y 3 del documento no seguía su criterio y que el #7 "abarata" (no "desbloquea") al #13.

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Corrige el orden de los niveles 2 y 3
```

**Qué salió:** reordenó las filas de los niveles 2 (6 → 9 → 8 → 7) y 3 (14 → 15 → 12 → 10 → 17 → 13 → 16 → 18 → 11 → 19) sin renumerar, y añadió encima de cada nivel el cálculo que justifica el orden.

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Tengo problema para entender el porque el hallazgo 3 de la falta de test es el que desbloquea toda su serie de hallazgos subsecuentes. ¿Realmente la falta de tests es bloqueo de todos esos hallazgos? ¿Por que?
```

**Qué salió:** el agente reconoció que la falta de tests no bloquea técnicamente nada; distinguió bloqueo real (solo #5, el refactor), "abarata" por norma TDD (#4, #8, #12), dependencia indirecta vía #5 y ninguna relación (frontend, C-1, C-2). Corrigió su afirmación anterior de que "sin tests no se puede tocar nada".
