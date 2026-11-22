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

**Modelo:** GPT-6.2 Sol Light
**Herramienta:** Codex

```
Estoy preparando una auditoría. No modifiques archivos ni ejecutes comandos que cambien el repositorio.

Desde la raíz de este proyecto:

1. Resume los últimos 5 commits: hash corto, mensaje y qué cambio relevante introdujo cada uno.
2. Busca en qué archivos aparece la palabra del dominio `comentario` (incluye variantes razonables como `comentarios` y `comment` si las hubiera).
3. Indica qué comando usaste para cada búsqueda y entrega las rutas encontradas agrupadas por `backend`, `frontend`, `tests` y otros directorios.

Quiero solo resultados verificables y concisos.
```

**Qué salió:** Funciono bien, me listo 3 commits.

## Prompt 2

**Modelo:** GPT-6.2 Sol Light
**Herramienta:** Codex

```
Audita únicamente la feature de comentarios de este blog. No modifiques código de la aplicación, tests, configuración ni datos. Tu único archivo editable es `docs/auditoria/comentarios.md`.

Alcance obligatorio: creación, edición, borrado y listado de comentarios, siguiendo el recorrido completo entre API/rutas, base de datos y frontend. No audites posts, autores, categorías, etiquetas ni el sistema en general salvo que una evidencia de comentarios los afecte directamente.

Investiga el código y escribe una auditoría priorizada en `docs/auditoria/comentarios.md`.

Para cada hallazgo confirmado, crea una fila de tabla Markdown con exactamente estas seis columnas:

1. Qué es — una frase; separa los problemas distintos en filas distintas.
2. Dónde está — archivo, línea o rango de líneas que lo demuestra.
3. Qué se rompe y quién lo nota — efecto concreto y persona/actor afectado.
4. Cuántos sitios hay que tocar — número contado y los archivos o capas considerados.
5. ¿Se deshace con un revert? — sí/no y, si no, explica por qué.
6. ¿Bloquea o desbloquea a otro hallazgo? — nombra el otro hallazgo o escribe “A ninguno”.

Ordena primero por precedencia: lo que desbloquea o abarata otros arreglos va antes. Dentro del mismo nivel, prioriza por daño dividido por radio de impacto. Si un problema tiene mucho daño pero baja precedencia, sepáralo en dos filas: una de contención inmediata y otra de arreglo de fondo.

No inventes evidencia, líneas, efectos ni cantidades. Si no puedes verificar una casilla, déjala vacía de forma intencional. Las propuestas que aún no estén medidas deben quedar fuera de la tabla, bajo una sección “Propuestas por medir”, indicando qué habría que medir.

Al terminar, añade una sección breve “Resultado del agente” con:
- Número total de hallazgos devueltos.
- Lista de archivos que abriste para verificarlos.

No completes las tres líneas personales de la Parte B: las escribiré yo después de comprobar manualmente tus resultados.

Primero explora y verifica; después escribe el archivo. Al finalizar, responde con un resumen muy corto de lo escrito y no realices ningún otro cambio.
```

**Qué salió:**
Me devovió la auditoria escrita en el archivo comentarios.md, en el archivo dejé solamente la tabla con los 19 hallazgos