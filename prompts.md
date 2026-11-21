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

**Modelo:** Haiku 5.5 1M xMedium
**Herramienta:** Claude Code

```
Realiza una auditoría centrada solo en la feature de comentarios del blog (crear, editar, borrar y listar comentarios), en sus tres capas: rutas de la API, tabla de la base de datos y lo que enseña la web. Quedan fuera posts, autores, categorías y etiquetas, aunque te ofrezcas a auditarlos. Si encuentras algo fuera de comentarios, anótalo en una línea aparte debajo de la tabla, sin fila.

Contrasta contra estas reglas de negocio: un comentario pendiente no se muestra en el detalle del post; el recuento de comentarios cuenta solo los aprobados; la respuesta pública no expone el correo de quien comenta. Indica también cómo se aprueba un comentario, si existe alguna forma.

Escribe el resultado en docs/auditoria/comentarios.md: una fila por hallazgo con seis casillas: qué es (una frase), dónde está (archivo y línea que lo demuestra), qué se rompe y quién lo nota, cuántos sitios hay que tocar (contados, no estimados), si se deshace con un revert (sí o no, y por qué no), y si bloquea o desbloquea a otro hallazgo (nombrándolo). Si no puedes dar la línea que lo demuestra, no escribas la fila.

Es una auditoría, no un arreglo. Ignora las secciones de Git, TDD y Definition of Done del CLAUDE.md: no escribas tests, no refactorices, no cambies ningún archivo salvo docs/auditoria/comentarios.md, no crees ramas (sigue en la actual), no hagas commits y no abras ningún PR ni uses gh. Al terminar, dime cuántos hallazgos devolviste.
```

**Qué salió:** (opcional, una línea) funcionó a la primera / tuve que insistir / me inventó una ruta que no existe.
