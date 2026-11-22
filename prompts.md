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

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
Audita la feature de comentarios del blog (solo esa): sus rutas en la API
(POST, PATCH y DELETE de comentarios y el listado que sale en GET /posts/:slug),
la tabla comentarios y lo que enseña la web (Comentarios.jsx y el recuento de
DetallePost.jsx). No audites posts, autores, categorías ni etiquetas.

Antes de escribir nada, lee backend/start/routes.ts entero, la migración y la
semilla de comentarios, y los componentes de la web. Contrasta cada hallazgo con
las seis reglas de negocio del README.

Escribe el resultado en docs/auditoria/comentarios.md, no en el chat. Una tabla,
una fila por hallazgo, con seis columnas: 1) qué es (una frase), 2) dónde está,
con el archivo y la línea que lo demuestra, 3) qué se rompe y quién lo nota,
4) cuántos sitios hay que tocar, contados, 5) si se deshace con un revert y, si
no, por qué, 6) si bloquea o desbloquea a otro hallazgo, nombrándolo. Ordénalos
por prioridad.

No afirmes nada que no hayas comprobado: levanta la API y prueba los endpoints
con curl sobre una copia de la base de datos, y marca cada hallazgo como
ejecutado o solo leído en el código. Si una fila no la puedes completar, déjala
a medias en lugar de rellenarla de memoria.

Debajo de la tabla, tres líneas: cuántos hallazgos devolviste y cuántos
comprobaste, el hallazgo del que no sabes decidir la prioridad y por qué, y algo
que afirmaste y no pudiste verificar.

No toques código, solo el documento.
```

**Qué salió:** funcionó a la primera: `docs/auditoria/comentarios.md` con la tabla y la Parte B.
