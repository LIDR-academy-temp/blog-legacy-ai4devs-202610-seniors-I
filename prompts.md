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
Vamos a realizar una auditoria de la feature "Comentarios" en este proyecto, de manera que analicemos si se cumplen las reglas de negocio descritas en CLAUDE.md en tres capas diferentes: en las rutas de la API, en las tablas de la base de datos, y en el front-end del blog.

Comprueba que los comentarios se pueden crear, editar, borrar y listar, y que para cada una de estas 4 acciones, se cumplen las reglas de negocio.

Una norma importante de cara a realizar la auditoría: un hallazgo sin la línea que lo demuestra, no es un hallazgo. Por lo tanto, para cada una de las reglas de negocio que no se cumplan, quiero que indiques dónde lo has visto (tanto en la API, como en la base de datos o el front-end).

Los resultados de la auditoría escríbelos en el archivo docs/auditoria/comentarios.md, y por cada hallazgo escribe una línea con lo siguiente:

1. Qué es: una frase; si necesitas más de una, es que hay varios hallazgos.
2. Dónde está: la línea que lo demuestra; sin esta casilla, la fila no se escribe.
3. Qué se rompa, y quién lo nota: es el efecto concreto y a quién le llega. "Nadie todavía" es una respuesta buena y válida.
4. Cuántos sitios hay que tocar: contando los sitios en los que habría que modificar el código para corregirlo.
5. ¿Se deshace con un revert?: Sí o no, y si es que no, por qué no.
6. ¿Bloquea o desbloquea otro hallazgo?: Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

**Qué salió:** Una auditoría con 24 hallazgos de cosas que no se están cumpliendo.


## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Vamos a revisar C-01, muéstrame la línea de código que guarda el email de una persona que comenta un post en el estado del front-end y que, por lo tanto, permite que cualquier visitante lo vea en las devtools.
```

**Qué salió:**

```
Es frontend/src/componentes/Comentarios.jsx:22:

19          fetch(API + '/posts/' + encodeURIComponent(slug))
20            .then((r) => r.json())
21            .then((d) => {
22              setComentarios(d.comentarios || [])

d.comentarios llega tal como lo devuelve la API, así que cada comentario trae su
autor_email. Esa línea guarda los objetos completos en el estado, y el correo se
puede ver en React DevTools.

Al revisarlo he visto además algo que se me pasó: hay un segundo sitio en el front
que guarda el correo en el estado, frontend/src/componentes/DetallePost.jsx:37:

37              setDatos(d)

Esa línea guarda la respuesta entera. Además de los correos de quienes comentan,
incluye los del autor del post (post.autor_email y autor.email, que son C-02 y
C-03). En la fila de C-01 solo cité Comentarios.jsx:22.

Esa línea guarda la respuesta entera. Además de los correos de quienes comentan,
incluye los del autor del post (post.autor_email y autor.email, que son C-02 y
C-03). En la fila de C-01 solo cité Comentarios.jsx:22.

¿Quieres que corrija la fila de C-01 en docs/auditoria/comentarios.md? Cambiaría dos
cosas:
- La columna «Dónde está» citaría las dos líneas del front como síntomas, no como
causa.
- La columna «Qué se rompe» diría que se ve en la pestaña Red, aunque el front no
guarde nada.
```

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Si, actualízalo.
```

**Qué salió:** Se ha actualizado el contenido de `docs/auditoria/comentarios.md` modificando la columna "Dónde está" y "Qué se rompe".


## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Muéstrame la línea de código que valida el hallazgo C-02.
```

**Qué salió:**

```
 La línea que demuestra C-02 es backend/start/routes.ts:167. Para entenderla hacen
falta dos líneas más de la API y una de la base de datos.

1. La consulta trae todas las columnas del post, backend/start/routes.ts:111:

111         "select * from posts where slug = ? and estado = 'publicado' limit 1",

2. La fila se devuelve sin filtrar, backend/start/routes.ts:117 y :167:

117       const post = posts[0]
...
166       return response.send({
167         post: post,

Entre la línea 117 y la 167 no se quita ningún campo de post. Lo que sale de la base
de datos es exactamente lo que recibe el cliente.

3. Por qué esa fila trae un correo,
backend/database/migrations/1769000000004_create_posts_table.ts:33:

30            // Denormalización a mano: el nombre y el correo del autor y el nombre
de la
31            // categoría, copiados dentro del post.
...
33            table.string('autor_email').nullable()

La tabla posts guarda una copia del correo del autor. Así que select * lo incluye.

Comprobado en vivo: al principio de la auditoría pedí GET
/posts/diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna a la API
levantada, y la respuesta traía post.autor_email: "marta.ruiz@corriente.es".
```

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Vayamos a C-04, muéstrame dónde se ve que incumple las reglas de negocio.
```

**Qué salió:**

```
El código de C-04 está en backend/start/routes.ts:

329       const nuevos: any = await db.rawQuery(
330         'select * from comentarios order by id desc limit 1'
331       )
332
333       return response.status(201).send({ comentario: nuevos[0] })

La línea 330 trae todas las columnas del comentario y la 333 las devuelve sin
filtrar. La respuesta contiene id, post_id, autor_nombre, autor_email, cuerpo,
estado, creado_en y padre_id.

Al revisarlo, la etiqueta [R6] que le puse a C-04 en el documento no se sostiene tal
como está.

Por qué no incumple R6 por sí solo. R6 (CLAUDE.md:148) prohíbe que la respuesta
pública de un post exponga el correo de quien comenta. Su objetivo es que nadie vea
el correo de otra persona. En el uso normal, el correo que devuelve el POST es el
mismo que quien comenta acaba de enviar en la línea 306
(request.input('autor_email')). Esa persona solo recupera su propio correo, y ningún
tercero ve nada.

Qué reglas sí incumple, comprobado línea a línea. Son las reglas de arquitectura de
CLAUDE.md, no las de negocio:
- CLAUDE.md:60: «Todo endpoint devuelve un transformer, nunca la fila cruda». La
  línea 333 devuelve la fila cruda.
- CLAUDE.md:62: «Tampoco salen campos internos (created_at, updated_at, estado, ids
  ajenos) si el contrato no los pide». La respuesta incluye estado: 'pendiente' y
  post_id.

Cuándo sí incumple R6: solo a través de C-05. La línea 330 no lee el comentario
recién insertado, sino el último que haya en la tabla. Si dos personas comentan a la
vez, la primera puede recibir el comentario de la segunda, con su correo. Ahí sí se
expone el correo de un tercero. Pero la línea que lo provoca es la 330 (C-05), no
la forma de la respuesta (C-04).

Corrección que propongo para la fila de C-04:
- Cambiar la etiqueta de [R6 · Crear] a [Arquitectura · Crear].
- En «Qué es», citar CLAUDE.md:60 y :62 como reglas incumplidas.
- Añadir que solo llega a incumplir R6 combinado con C-05.

Revisé de la misma forma C-06, la respuesta de editar (routes.ts:389-392), y le pasa
lo mismo. Quien edita tiene que enviar su correo exacto para que la edición se
acepte (línea 379), así que solo recibe el suyo. También habría que quitarle la
etiqueta R6.

¿Aplico la corrección a C-04 y C-06? Si las quito de R6, también hay que actualizar
la matriz de la sección 1, porque en la fila Crear cita C-04 como incumplimiento de
R6.
```


## Prompt 6

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
❯ Si, aplica la corrección tanto en C-04 como en C-06
```

**Qué salió:** Ha modificado el archivo `docs/auditoria/comentarios.md` para indicar que C-04 y C-06 incumplen las normas de Arquitectura, y no las reglas de negocio.

Me he quedado sin tiempo para continuar con la auditoría.