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

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
Realiza una Auditoría de la feature de "comentarios" y déjala escrita en docs/auditoria/comentarios.md, no en el chat.

Una feature es una funcionalidad del blog de punta a punta: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella. Este blog tiene cinco (posts, comentarios, autores, categorías y etiquetas), esta Auditoría es solo de comentarios: crear, editar, borrar y listar comentarios, en esas tres capas.

## Reglas:
- Solo esa feature. No el sistema entero.
- El formato no es negociable.

## Formato:
Una fila por hallazgo, con estas seis casillas:
- Qué es: Una frase. Si necesitas tres, es que hay dos hallazgos.
- Dónde está: con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.
- Qué se rompe, y quién lo nota: El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
- Cuántos sitios hay que tocar: Contados, no estimados. "Toca varios sitios" no sirve: si son tres, se escriben los tres.
- ¿Se deshace con un revert?: Sí o no, y si es que no, por qué no. Si toca algo que ya está guardado o algo que otros ya están leyendo.
- ¿Bloquea o desbloquea a otro hallazgo?: Nombrando al otro hallazgo.
```

**Qué salió:** (opcional, una línea) funcionó a la primera, pero aunque no le pedí que priorizara, al parecer leyó el README.md y usó el contexto de la tarea.

## Prompt 2

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
la C23 no es una regla rota ni una problema de seguridad, no es un Daño, así que la eliminamos y sus referencias en otras tareas.
```

**Qué salió:** (opcional, una línea) funcionó a la primera.

## Prompt 3

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
quita también la C22 y la C21
```

**Qué salió:** (opcional, una línea) funcionó a la primera.

## Prompt 4

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
Reglas de negocio:
Esto es lo que el sistema debe cumplir. Es el contraste de la auditoría: sin una regla escrita, un hallazgo no se debe tener en cuenta.

- Un post en borrador no aparece en listados, relacionados, recientes ni búsqueda, ni siquiera para su autor.
- Un comentario pendiente no se muestra en el detalle del post.
- slug es único entre posts.
- Los relacionados nunca incluyen el propio post, y salen como mucho N.
- El recuento de comentarios cuenta solo los aprobados.
- La respuesta pública de un post no expone el correo del autor ni el de quien comenta.

Revisa los resultados y elimina los hallazgos que no estén relacionados con las reglas.
```

**Qué salió:** (opcional, una línea) funcionó a la primera.

## Prompt 5

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
recupera los hallazgos relacionados con la seguridad en el borrado
```

**Qué salió:** (opcional, una línea) funcionó a la primera.
