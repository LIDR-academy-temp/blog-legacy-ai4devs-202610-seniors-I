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

`````
Guarda cada entrada mía en el archivo prompts.md; fíjate en que ## Prompt 1 ya está relleno con un prompt de ejemplo. Haz lo mismo con Prompt 2, etc. Guarda cada entrada, sean comandos o prompts, tal cual la escribo, sin corregir nada. Pon **Modelo:** Opus 5.5 y **Herramienta:** Claude Code. Si mi entrada lleva ``` dentro, usa ```` como delimitador. Solo en esta sesión. Al terminar tu respuesta a cada entrada, rellena su "Qué salió:" con una línea corta y objetiva: qué produjiste o qué archivos tocaste, y si funcionó a la primera, falló o se quedó a medias. No te pongas nota ni valores la calidad. Si mi siguiente entrada corrige o repite la anterior, añade a ese "Qué salió:" anterior "tuve que insistir: <motivo>". Empieza con este y sobrescribe el ejemplo
`````

**Qué salió:** Sobrescribí el ejemplo de prompts.md con este prompt como Prompt 1; funcionó a la primera.

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Vamos a auditar la feature de comentarios de este blog. Es una auditoría, no un arreglo: ignora las secciones de Git, TDD y Definition of Done de CLAUDE.md. No modifiques, crees ni borres ningún archivo salvo docs/auditoria/comentarios.md y prompts.md. No escribas tests, no crees ramas, no hagas commits ni uses gh.

Alcance: solo comentarios (crear, editar, borrar y listar) en sus tres capas: las rutas de backend/start/routes.ts que los tocan (incluido el listado dentro de GET /posts/:slug y los recuentos de comentarios), la tabla comentarios (migración y semilla) y lo que enseña la web (Comentarios.jsx, DetallePost.jsx y donde se pinte el recuento). Lo que veas fuera de comentarios va en una línea debajo de la tabla, sin fila.

Contrasta con las 6 reglas de negocio del README. Recorre una capa cada vez y junta en una sola fila los defectos que nacen de la misma decisión, aunque aparezcan en varias líneas.

Antes de escribir una fila, clasifica cada candidato: hecho (se ve abriendo el archivo), riesgo (se deriva de un hecho) u opinión (no se puede comprobar). Las opiniones no entran en la tabla: van en una sección "Propuestas sin medir", con lo que habría que medir al lado.

Tabla en docs/auditoria/comentarios.md, una fila por hallazgo, con estas columnas: # | Qué es | Dónde (archivo:línea) | Qué se rompe y quién lo nota | Sitios (contados) | ¿Revert? (sí/no y por qué) | Bloquea o desbloquea (a cuál). Celdas cortas, unas 15 palabras como mucho: la tabla tiene que caber en una pantalla. Sin archivo:línea la fila no se escribe. Si una casilla no la puedes responder con algo que hayas leído o ejecutado, déjala en blanco; no la rellenes de memoria. "Nadie todavía" y "A ninguno" son respuestas válidas.

Orden: primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes); dentro de cada nivel, daño dividido por radio. No ordenes por gravedad. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, pártelo en dos filas: una contención pequeña que corta el daño hoy y el arreglo de fondo en su turno.

Para comprobar, lee el código y usa git, grep y consultas de solo lectura. Contra la API (localhost:3401), solo GET. Nada que escriba en la base: si un hallazgo necesita un POST, PATCH o DELETE para verse, escribe en el chat el curl exacto y no lo ejecutes; lo lanzo yo.

Al terminar, dime en el chat cuántos hallazgos devolviste (filas de la tabla y propuestas, por separado) y, por cada fila, en una línea, el archivo:línea que tengo que abrir para comprobarla.
```

**Qué salió:** Creé docs/auditoria/comentarios.md con 9 filas, 5 propuestas sin medir y "Las tres líneas" vacías; API sin levantar, verificado con lectura de código y SQLite en solo lectura. Funcionó a la primera. Tuve que insistir: la tabla no se leía (celdas largas, varias rutas por casilla).

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
La tabla de docs/auditoria/comentarios.md no se lee: celdas largas, varias rutas por casilla y todo mezclado. Rehaz solo la forma; no cambies los hallazgos, el orden ni lo que afirma cada casilla.

- Encabezados: # | Qué es | Dónde | Qué se rompe · quién | Sitios | ¿Revert? | Bloquea
- Celdas de 8 palabras como mucho. Lo que no quepa va a una sección "Notas" debajo de la tabla, numerada con el # de su fila.
- Dónde: una sola referencia, la línea que demuestra el hallazgo, con el nombre corto del archivo (routes.ts:153, Comentarios.jsx:39, migración:18). Las demás líneas van a Notas.
- Qué se rompe · quién: el efecto y quién lo nota, separados por " · ".
- Sitios: solo el número.
- ¿Revert?: "Sí" o "No: <motivo en 4 palabras>".
- Bloquea: solo "Desbloquea #N", "Bloqueado por #N" o "A ninguno".
- No alinees las columnas con espacios de relleno: cada fila, lo más corta posible.
- Lo mismo para la tabla de propuestas. El párrafo de fuera de alcance pásalo a una lista, un punto por línea.
```

**Qué salió:** Rehíce la forma de docs/auditoria/comentarios.md: tabla compacta, sección Notas, fuera de alcance en lista y propuestas compactadas; funcionó a la primera.
