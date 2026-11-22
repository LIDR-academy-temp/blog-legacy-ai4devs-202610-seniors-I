# Auditoría priorizada · feature **comentarios**

**Alcance:** crear, editar, borrar y listar comentarios, en sus tres capas.

- **API:** `POST /posts/:slug/comentarios`, `PATCH /comentarios/:id` y `DELETE /comentarios/:id`, más
  la lista de comentarios de `GET /posts/:slug` y los recuentos `num_comentarios` de `GET /posts`
  y de `GET /posts/:slug/relacionados`.
- **Base de datos:** la tabla `comentarios`, su migración y su semilla.
- **Web:** `Comentarios.jsx`, el recuento de `DetallePost.jsx` y los de `ListaPosts.jsx` y
  `PostsRelacionados.jsx`.

El resto del sistema queda fuera.

**Cómo se ordena:**

1. **Precedencia primero.** Un hallazgo va antes que otro si lo desbloquea o lo abarata.
   Los niveles salen del grafo de la última columna: nivel 0 = nada lo bloquea.
2. **Dentro de cada nivel, daño ÷ radio.** Daño D, de 1 a 5:
   - 5 = pérdida irreversible de datos para cualquiera;
   - 4 = datos personales expuestos, o la acción de otro usuario a tu alcance;
   - 3 = contenido legítimo que desaparece, o números que mienten al lector;
   - 2 = contrato incoherente que nota un cliente de la API;
   - 1 = cosmético.

   Radio R = los sitios contados en la cuarta casilla. Si hay empate, va antes el que desbloquea más.
3. **Partidos en dos.** Los tres hallazgos de más daño (#4, #1 y #5) tienen el arreglo de fondo
   en lo más bajo del grafo. Por eso se parten en una **contención**, que corta el daño hoy, y un
   **arreglo de fondo** (#18, #11 y #15).

**Comprobación:**

- Todas las líneas citadas se han abierto en el archivo.
- Los recuentos de #6 y #14 se han medido con SQL contra `backend/tmp/db.sqlite3`, con la
  semilla cargada.
- La carrera de #7 y el desfase de fecha de #12 se deducen leyendo el código. **No se han
  reproducido.**

## Tabla

| Qué es | Dónde está (la línea que lo demuestra) | Qué se rompe, y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro? |
|---|---|---|---|---|---|
| **— Nivel 0 —** | | | | | |
| **#1 · Contención.** `GET /posts/:slug` devuelve el correo de cada persona que comenta. | `backend/start/routes.ts:152-153`: `"select * from comentarios where post_id = ? and estado = 'aprobado' …"`, que sale tal cual en `:171` (`comentarios: comentarios`). | Cualquiera que abra el detalle de un post en las devtools o con `curl` se lleva nombre y correo de quien comentó. En la semilla son, entre otros, `ana.cortes@ejemplo.es` y `marta.ruiz@corriente.es`. Incumple la regla 6. Nadie lo ha denunciado todavía. | 1: la consulta de `:152-153`, listando las columnas sin `autor_email`. Más 1 test con `assert.notProperty`. | Sí, el código. Los correos ya servidos no vuelven. | **Abarata #5 y #15**: deja de publicar la credencial que usan DELETE y PATCH. El arreglo de fondo es **#11**. |
| **#2 ·** Ningún comportamiento de comentarios está caracterizado: el único test afirma un campo. | `backend/tests/functional/comentarios.spec.ts:41-42`: `assertStatus(201)` y `assert.equal(…autor_nombre, 'Lucía Ferrer')`. | Cualquier cambio en las rutas de comentarios rompe a ciegas: la suite sigue verde. Hoy no lo nota nadie. Lo notará quien refactorice. | 1 archivo con 6 comportamientos: POST `:295`, PATCH `:366`, DELETE `:339`, lista del detalle `:152`, recuento de `/posts` `:75` y recuento de relacionados `:222`. | Sí (solo añade tests). | **Desbloquea #6 a #20**: todo lo que cambia código sin ser contención. |
| **#3 ·** La web suma 1 al recuento al enviar un comentario que nace `pendiente` y no se muestra. | `frontend/src/componentes/Comentarios.jsx:39`: `setTotal(total + 1)`. El aviso de `:38` dice «pendiente de moderación». | Quien comenta ve «3 comentarios» encima de una lista de 2. El lector que recarga ve otra cifra. Incumple la regla 5. | 1 (`Comentarios.jsx:39`). | Sí. | A ninguno. Lo absorbe **#19** cuando llegue. |
| **#4 · Contención.** La clave de moderación de DELETE está escrita en el código y publicada en el repo. | `backend/start/routes.ts:25`: `const SECRETO_SESION = 'dev-secret-cambiar'`, y se usa en `:351`: `clave === SECRETO_SESION`. | Cualquiera que lea el repo, que está forkeado en público, puede borrar **cualquier** comentario con `x-clave-moderacion: dev-secret-cambiar`. El borrado es físico (`:359`). Nadie lo ha notado todavía. | 3: `routes.ts:25` (leer de env), `backend/start/env.ts` (declarar la variable) y `backend/.env.example`. Además hay que rotar la clave en cada `.env` desplegado. | El código sí. **La clave no**: sigue en el historial de git y en los forks, así que hay que rotarla. Un revert volvería a poner en servicio una clave pública. | **Desbloquea #5**: cerrar la puerta del correo no sirve mientras la clave pública borra igual. El arreglo de fondo es **#18**. |
| **— Nivel 1 —** | | | | | |
| **#5 · Contención.** DELETE da por autor a quien envíe el mismo correo que el comentario. | `backend/start/routes.ts:350-353`: `request.input('autor_email', '')` comparado con `comentario.autor_email`. | Quien conozca o raspe un correo (#1) borra comentarios ajenos. La web no tiene botón de borrar, así que hoy no lo nota nadie de la web. Lo explota cualquier cliente de la API. | 1: `routes.ts:352-355`. Mientras no exista #15, solo borra quien traiga la clave de moderación. | Sí. | Va después de **#4** y de **#1**. El arreglo de fondo es **#15**. |
| **#6 ·** `comentarios.estado` es texto libre y la semilla mezcla `'aprobado'`, `'Aprobado'` y `'ok'`. | Migración `1769000000005_create_comentarios_table.ts:18`: `table.string('estado')`, sin CHECK. Semilla `blog_seeder.ts:510` y `:575` (`'Aprobado'`) y `:522` (`'ok'`). Medido: 4 `aprobado`, 2 `Aprobado`, 1 `ok` y 2 `pendiente`. | Tres comentarios aprobados no salen en el detalle: los de Rubén Lima, Silvia Peña y Paula Merino. Lo nota quien los escribió. El post 6 enseña 0 comentarios teniendo uno aprobado. | 1 migración de datos nueva, con la normalización y el CHECK (en SQLite el CHECK obliga a reconstruir la tabla). Son 3 filas más de la semilla si también se limpia. | **No**: pasar `'Aprobado'` y `'ok'` a `'aprobado'` pierde el valor original, y `down()` no puede saber qué fila era `'ok'`. Hace falta copia previa. | **Desbloquea #14** (con datos limpios las dos reglas coinciden) y **#20** (qué valor escribe la aprobación). |
| **#7 ·** POST devuelve el último comentario de la tabla, no el que acaba de insertar. | `backend/start/routes.ts:329-331`: `'select * from comentarios order by id desc limit 1'`. | Si dos POST se cruzan entre el `await` del insert y el del select, uno recibe el comentario del otro, con el correo incluido. Nadie lo ha notado todavía: con poco tráfico no se cruzan. **No reproducido.** | 1: `routes.ts:324-331`, usando el id que devuelve el insert. | Sí. | A ninguno. |
| **#8 ·** Borrar un comentario deja sus respuestas huérfanas, y la web las esconde sin avisar. | `backend/start/routes.ts:359`: `'delete from comentarios where id = ?'`, sin tocar a los hijos. Migración `:20`: `padre_id` sin FK. `Comentarios.jsx:50`: `filter((c) => !c.padre_id)` solo pinta raíces y sus hijos directos. | Si se borra el comentario 1 (Ana Cortés), la respuesta 2 (Marta Ruiz) desaparece de la web, pero sigue en la tabla y en el recuento. Lo nota Marta. | 1 (`routes.ts:359`), o 2 si se decide FK con cascada (una migración nueva). Requiere una decisión de producto: cascada o marcar «[borrado]». | El código sí. Si se elige cascada, las respuestas borradas no vuelven. | A ninguno. |
| **#9 ·** PATCH y DELETE autorizan al autor con dos reglas distintas escritas a mano. | `backend/start/routes.ts:352-353` (DELETE: pasa a minúsculas, recorta y admite la clave) frente a `:379` (PATCH: `comentario.autor_email !== email`, con mayúsculas, sin clave y solo si está `pendiente`). | Quien escribió `Marta@…` puede borrar su comentario pero no editarlo, y un moderador no puede corregir una errata. Nadie lo nota desde la web, porque no expone ninguna de las dos acciones. | 2 (`:352-353` y `:379`) → una sola política en el servicio. | Sí. | **Abarata #15 y #18**: con una sola política, cambiar la identidad o el moderador es un sitio y no dos. |
| **#10 ·** La validación del POST son `if`s a mano con cuatro formas de error distintas, y una responde 200. | `backend/start/routes.ts:310-321`: `400 {error}`, `422 {message}`, `400 {errores:[…]}` y, en `:320`, `response.send({ ok: false, motivo: … })` con 200. | Un cliente de la API o la monitorización ven 200 en un envío fallido. La web lo disimula con `d.error \|\| d.message \|\| d.motivo` (`Comentarios.jsx:44`). | 4: `routes.ts:310-321`, el nuevo `app/validators/comentario.ts`, `app/exceptions/handler.ts` y `Comentarios.jsx:44`. | Sí, pero cambia el contrato que lee `Comentarios.jsx:44`: el revert tiene que ser conjunto, back y front. | **Desbloquea #13, #16 y #17**: los tres son una regla más del mismo validador. |
| **#11 · Arreglo de fondo de #1.** Todas las respuestas de comentarios son filas crudas del `select *`, sin transformer. | `backend/start/routes.ts:171` (`comentarios: comentarios`), `:333` (`{ comentario: nuevos[0] }`) y `:392` (`{ comentario: actualizados[0] }`). | Además del correo, salen `estado` y `padre_id`, que el contrato no pide. Cualquier columna que se añada saldrá sola. Hoy no lo nota nadie. | 4: las tres respuestas citadas y el nuevo `app/transformers/comentario_transformer.ts`. | Sí. | A ninguno. Sustituye la contención #1. |
| **#12 ·** `creado_en` se guarda en UTC sin zona y la web recorta la fecha a 10 caracteres. | `backend/start/routes.ts:323`: `new Date().toISOString().slice(0, 19).replace('T', ' ')`. `Comentarios.jsx:59` y `:68`: `String(c.creado_en).substring(0, 10)`. | Un comentario de las 00:30 en Madrid sale fechado el día anterior. Nadie lo ha notado todavía. **No reproducido.** | 3 (`routes.ts:323`, `Comentarios.jsx:59` y `:68`). | No del todo: las filas escritas con el formato nuevo quedan mezcladas con las viejas en UTC. | A ninguno. |
| **— Nivel 2 —** | | | | | |
| **#13 ·** PATCH acepta un cuerpo de 1 carácter, saltándose el mínimo de 5 que exige el POST. | `backend/start/routes.ts:384`: `if (!cuerpo)`, frente a `:319`: `String(cuerpo).length < 5`. | Un comentario se puede editar a «a» y saltarse la regla de creación. Nadie lo nota todavía. | 1 (`routes.ts:384` → reutilizar el validador de #10). | Sí. | Va después de **#10**. |
| **#14 ·** «Aprobado» tiene dos definiciones: el detalle usa `= 'aprobado'` y los recuentos `<> 'pendiente'`. | `backend/start/routes.ts:153` (`estado = 'aprobado'`) frente a `:75-76` y `:223` (`estado <> 'pendiente'`). Medido en la semilla: el post 1 lista **4** y su detalle muestra **2**; el post 6 lista **1** y muestra **0**. | El lector ve «4 comentarios» en la portada y encuentra 2 al entrar. Un futuro `'rechazado'` o `'spam'` contaría como aprobado. Incumple la regla 5. | 3 (`:75-76`, `:153` y `:223`). | Sí. | Va después de **#6**. **Desbloquea #19**: el número que dé la API ya será el correcto. |
| **#15 · Arreglo de fondo de #5.** La única identidad del autor de un comentario es su correo, que no es secreto. | `backend/start/routes.ts:350-353` (DELETE) y `:378-379` (PATCH). | Sin una identidad real no se puede devolver al autor el derecho a borrar ni a editar sin reabrir #5. Lo notará el autor que quiera retirar su comentario. | 4: `:350-353`, `:378-379`, `:333` (devolver un token al crear) y 1 migración con la columna del token. Requiere una decisión de producto: token por comentario o login. | **No**: un revert de la migración tira la columna y los tokens ya emitidos, y los autores pierden el acceso. | Va después de **#9**. **Cierra #5**: permite retirar la contención. |
| **#16 ·** `padre_id` se guarda sin comprobar que exista, que sea del mismo post ni que sea una raíz. | `backend/start/routes.ts:308` (`request.input('padre_id', null)`), insertado en `:326`. Migración `:20`: sin FK. | Un cliente de la API puede crear respuestas que la web no pinta nunca: las de otro post, las de un id inexistente y las respuestas a respuestas, por cómo filtra `Comentarios.jsx:50` y `:63-64`. La web no envía `padre_id` (`:33`). Nadie lo nota todavía. | 2 (la regla en el validador de #10 y la consulta de existencia). | Sí. | Va después de **#10**. |
| **#17 ·** El correo se da por válido si contiene una `@`. | `backend/start/routes.ts:316`: `String(email).indexOf('@') === -1`. | Se aceptan `a@` o `@`. Nadie lo nota todavía. | 1 (`routes.ts:316` → `vine.string().email()`). | Sí. | Va después de **#10**. |
| **— Nivel 3 —** | | | | | |
| **#18 · Arreglo de fondo de #4.** Ser moderador es conocer una clave compartida, sin usuario ni rastro. | `backend/start/routes.ts:349-351`: `request.header('x-clave-moderacion')` comparada con una constante. La tabla `users` no tiene columna de rol (`1761885935168_create_users_table.ts:8-14`). | Aun con la clave en env, no se sabe quién borró qué, y revocar a un moderador obliga a rotar la clave a todos. Lo notará el primero que tenga que investigar un borrado. | 3: `routes.ts:349-351`, una migración con el rol en `users` y el `middleware.auth()` en la ruta. | No limpio: un revert de la migración tira los roles asignados. | Va después de **#9**. **Desbloquea #20**. |
| **#19 ·** La web lleva dos recuentos de comentarios independientes y pide el post dos veces. | `DetallePost.jsx:30` y `:38` (`setNumComentarios(…length)`) frente a `Comentarios.jsx:11` y `:23`. `Comentarios.jsx:19` vuelve a hacer `fetch(API + '/posts/' + …)`. | La cabecera del artículo (`DetallePost.jsx:71`) y la sección (`Comentarios.jsx:54`) dejan de coincidir en cuanto alguien comenta. Lo ve cualquier lector. | 7: `DetallePost.jsx:30`, `:38` y `:71`; `Comentarios.jsx:11`, `:23` y `:54`; y exponer `num_comentarios` en el detalle (`routes.ts:166-174`). | Sí. | Va después de **#14**. Absorbe **#3**. |
| **— Nivel 4 —** | | | | | |
| **#20 ·** Ninguna ruta de la API cambia `estado`, así que un comentario enviado desde la web no puede pasar a aprobado. | `backend/start/routes.ts:388`: el único `update` de comentarios es `'update comentarios set cuerpo = ? where id = ?'`. El POST lo fija a `'pendiente'` en `:326`. | Si no existe un panel de moderación fuera de este repo, todo comentario nuevo queda invisible para siempre, y lo nota cada persona que comenta. Si existe, no lo nota nadie. **No verificable desde el repo.** | 4: la ruta nueva en `routes.ts`, el controlador, el servicio y el validador del estado. | Sí el código. Los comentarios aprobados con él no se desaprueban con el revert. | Va después de **#6** (qué valor escribir) y de **#18** (quién puede aprobar). |

### Daño ÷ radio usados para ordenar

| # | D | R | D/R | Nivel |
|---|---|---|---|---|
| 1 | 4 | 1 | 4.0 | 0 |
| 2 | 3 | 1 | 3.0 | 0 |
| 3 | 2 | 1 | 2.0 | 0 |
| 4 | 5 | 3 | 1.67 | 0 |
| 5 | 4 | 1 | 4.0 | 1 |
| 6 | 3 | 1 | 3.0 | 1 (empata con #7 y #8, desbloquea más) |
| 7 | 3 | 1 | 3.0 | 1 |
| 8 | 3 | 1 | 3.0 | 1 (tras #7: necesita una decisión) |
| 9 | 2 | 2 | 1.0 | 1 |
| 10 | 2 | 4 | 0.5 | 1 (empata con #11, desbloquea tres) |
| 11 | 2 | 4 | 0.5 | 1 |
| 12 | 1 | 3 | 0.33 | 1 |
| 13 | 2 | 1 | 2.0 | 2 |
| 14 | 3 | 3 | 1.0 | 2 (desbloquea #19) |
| 15 | 4 | 4 | 1.0 | 2 |
| 16 | 2 | 2 | 1.0 | 2 |
| 17 | 1 | 1 | 1.0 | 2 |
| 18 | 3 | 3 | 1.0 | 3 |
| 19 | 2 | 7 | 0.29 | 3 |
| 20 | 4? | 4 | 1.0? | 4 (el daño depende de algo que no se puede verificar) |

**Nota sobre #4:** el comentario de `routes.ts:21-24` dice que la clave «firma las sesiones».
**Es falso**: las sesiones y el cifrado usan `APP_KEY` (`config/encryption.ts:16` y
`start/env.ts:22`). La constante solo sirve para moderar. Por eso la contención no toca sesiones.

## Propuestas sin medir (fuera de la tabla)

Nadie las ha medido, así que no entran en la prioridad. Cada una lleva al lado lo que habría que medir.

| Propuesta | Qué habría que medir antes |
|---|---|
| Índice en `comentarios.post_id`. La migración `:14` no lo crea, y en la semilla `EXPLAIN QUERY PLAN` da `SCAN c`. | La latencia p95 de `GET /posts` y de `/relacionados` con un volumen realista de comentarios. Con 9 filas el escaneo no se nota. |
| Quitar la segunda petición del detalle, la de `Comentarios.jsx:19`. | Cuántas peticiones y cuánto tiempo hasta pintar los comentarios en una vista de detalle, en red lenta. |
| Limitar la frecuencia de envío y añadir antispam en el POST (la semilla ya trae spam: el comentario 7). | Los POST por hora y por IP, y qué porcentaje de pendientes acaba borrado. |
| Longitud máxima de `cuerpo` y `autor_nombre`. Hoy solo los limita el límite por defecto del bodyparser JSON, que `config/bodyparser.ts:29-41` no fija. | La distribución real de longitudes de los comentarios guardados. |
| FK `post_id → posts` con cascada. | Cuántos comentarios huérfanos hay en producción: `select count(*) from comentarios where post_id not in (select id from posts)`. |
| Borrado lógico en vez de físico (`routes.ts:359`). | Cuántos borrados se reclaman o se quieren deshacer. |
| Paginar los comentarios del detalle. | El máximo y el p95 de comentarios aprobados por post. |

**Comprobado sin hallazgo:** el cuerpo del comentario se pinta como texto JSX
(`Comentarios.jsx:61`, `{c.cuerpo}`), no con `dangerouslySetInnerHTML`, así que por ahí no hay XSS.

**Visto y fuera de alcance:** son de otras features y no se auditan aquí.

- `post.autor_email` sale en el detalle (`routes.ts:167`, feature posts).
- El cuerpo del post se inyecta sin sanear (`DetallePost.jsx:92`, posts).
- CORS con `credentials: true` (sistema).

## Parte B: las tres líneas

1. **Hallazgos que devolvió el agente:** 20 filas, 3 de ellas partidas en contención y fondo
   (#1/#11, #4/#18 y #5/#15). **Comprobados por mí abriendo el archivo:** 5
2. **El hallazgo del que no supe decidir la prioridad, y por qué:** #20. Ninguna ruta aprueba comentarios, y su daño depende de si existe un panel de moderación fuera del repo.
3. **Algo que el agente afirmó y no pude verificar:** "10: La validación del POST son `if`s a mano con cuatro formas de error distintas, y una responde 200" No pude comprobar cual responde 200.
