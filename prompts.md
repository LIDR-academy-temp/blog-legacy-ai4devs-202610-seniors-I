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
Produce la auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md, no en el chat. Una feature es una funcionalidad del blog de punta a punta: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella. Este blog tiene cinco (posts, comentarios, autores, categorías y etiquetas), y la tuya es comentarios: crear, editar, borrar y listar comentarios, en esas tres capas. Solo esa feature. No el sistema entero. El orden debe resperar la precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño dividido por el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno. Y las propuestas que nadie ha medido todavía van fuera de la tabla, con lo que habría que medir escrito al lado.
El formato no es negociable: una fila por hallazgo, con estas seis casillas.

  1. Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

  2. Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

  3. Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

  4. Cuántos sitios hay que tocar. Contados, no estimados.

  5. ¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

  6. ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

**Qué salió:** (opcional, una línea) funcionó a la primera / tuve que insistir / me inventó una ruta que no existe.
```
He escrito la auditoría en docs/auditoria/comentarios.md. Tiene 20 filas con las seis casillas, de las que 3 son hallazgos partidos en contención y arreglo de fondo. No he hecho commit.

Cómo está ordenada:
- Precedencia: los niveles 0 a 4 salen de la última columna. Un hallazgo va antes si desbloquea o abarata a otro.
- Dentro de cada nivel: daño (de 1 a 5) dividido por el número de sitios contados en la cuarta casilla. Hay una tablita con los números debajo de la tabla, para que el orden se pueda discutir.
- Partidos en dos: la clave de moderación escrita en el código y publicada en el repo (#4, se arregla de fondo en #18), el correo de quien comenta que sale en el detalle del post (#1, se arregla en #11) y el borrado autorizado solo con un correo (#5, se arregla en #15).
- Lo primero de la lista: dejar de devolver el correo de quien comenta (1 sitio) y escribir los tests de caracterización, que desbloquean de #6 a #20.

Lo que he comprobado y lo que no:
- Todas las líneas citadas las he abierto en el archivo.
- He medido con SQL sobre la base de la semilla la diferencia entre la portada y el detalle: el post 1 dice 4 comentarios en la portada y muestra 2 al entrar; el post 6 dice 1 y muestra 0.
- Dos cosas las he deducido leyendo el código, sin reproducirlas, y lo pone en sus filas: que dos envíos simultáneos pueden devolver a uno el comentario del otro (#7) y que la fecha puede salir con un día de desfase (#12).
- El comentario de routes.ts:21-24 dice que esa clave firma las sesiones, y no es cierto: las sesiones usan APP_KEY.

Siete propuestas que nadie ha medido van fuera de la tabla, cada una con lo que habría que medir antes. Entre ellas, un índice en post_id, limitar la frecuencia de envío y quitar la petición duplicada del detalle.
```