# Auditoría — feature «Comentarios»

Fecha: 2026-11-21 · Rama auditada: `auditoria-las` (`1c36a09`) · Alcance: API (`backend/start/routes.ts`),
base de datos (`backend/database/`) y front (`frontend/src/`).

Reglas de negocio de referencia (CLAUDE.md): **R1** borrador oculto · **R2** comentario pendiente
oculto en el detalle · **R3** slug único · **R4** relacionados (no aplica a comentarios) · **R5** el
recuento cuenta solo aprobados · **R6** la respuesta pública de un post no expone correos.

**Cómo se verificó.** Lectura de código, consultas directas a `backend/tmp/db.sqlite3` y peticiones
a la API levantada en `localhost:3401` con la semilla cargada. Evidencias en vivo:

- `GET /posts/diez-tendencias-…` → `post.autor_email`, `autor.email` y `comentarios[].autor_email`
  (`ana.cortes@ejemplo.es`, `marta.ruiz@corriente.es`) salen en el cuerpo de la respuesta.
- Post 1: el listado cuenta **4** comentarios y el detalle muestra **2**. Post 6
  (`que-mide-de-verdad-…`): el listado cuenta **1** y el detalle muestra **0**.

**Cómo leer la columna «¿Se deshace con un revert?»**: si la *corrección*, una vez aplicada, se
puede deshacer con `git revert` sin dejar rastro (datos, secretos o efectos fuera del repo).

---

## 1. Matriz acción × regla × capa

| Acción | API | Base de datos | Front |
|---|---|---|---|
| **Crear** | Existe (`routes.ts:295`). R1 ✅ (`routes.ts:297` exige `publicado`). R6 ❌ C-05. Respuesta sin transformer C-04. R3 ❌ C-17. Validación incoherente C-22, `padre_id` sin validar C-16 | Sin FK ni CHECK: C-12, C-16 | Existe (`Comentarios.jsx:28`). R5 ❌ C-13. No permite responder C-21 |
| **Editar** | Existe (`routes.ts:366`). Respuesta sin transformer C-06. R1 ❌ C-24. Regla de longitud saltada C-18. Autorización C-07, C-19 | — | **No existe** C-20 |
| **Borrar** | Existe (`routes.ts:339`). R1 ❌ C-24. Autorización C-07, C-08, C-19. Respuestas huérfanas C-15 | `padre_id` sin FK C-15 | **No existe** C-20 |
| **Listar** | Sin endpoint propio: va dentro de `GET /posts/:slug` (`routes.ts:152`). R2 ✅ (`routes.ts:153`). R6 ❌ C-01, C-02, C-03. R5 ❌ C-09, C-10, C-11 | `estado` texto libre C-12 | R2 ✅ (solo pinta lo que trae la API). R5 ❌ C-09, C-10, C-13, C-14 |

Tests: ninguna celda de esta matriz está cubierta (C-23).

---

## 2. Hallazgos

Orden: primero lo que filtra datos o deja actuar a quien no debe, después los recuentos, después el resto.

| ID | Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea otro hallazgo? |
|---|---|---|---|---|---|---|
| C-01 | **[R6 · Listar]** El detalle del post devuelve el correo de cada persona que comenta. | **Causa (API):** `backend/start/routes.ts:153` (`select * from comentarios`) y `:171` (se devuelve tal cual). **Síntomas (front, no son la causa):** `frontend/src/componentes/Comentarios.jsx:22` (`setComentarios(d.comentarios \|\| [])`) y `frontend/src/componentes/DetallePost.jsx:37` (`setDatos(d)`, la respuesta entera) guardan los objetos con `autor_email` en el estado, sin pintarlo. BD: `backend/database/migrations/1769000000005_create_comentarios_table.ts:16` (la columna es correcta; el fallo es exponerla) | El correo ya está en la respuesta HTTP: cualquier visitante lo ve en la pestaña **Red** de las devtools o con `curl` (`make fuga`), guarde el front lo que guarde. React DevTools solo es una segunda forma de verlo. Lo sufren quienes comentan: sus correos quedan a la vista de bots de spam. En la interfaz no lo nota nadie. | 1 (proyección/transformer de comentarios en el detalle) | Sí. Pero los correos que ya se han servido no se pueden recuperar. | Desbloquea C-07: es lo que le da a cualquiera el correo que hace falta para borrar. |
| C-02 | **[R6 · Listar]** La respuesta que lista los comentarios también expone el correo del autor del post en `post.autor_email`. | API: `backend/start/routes.ts:111` (`select * from posts`) y `:167`. BD: `backend/database/migrations/1769000000004_create_posts_table.ts:33` (copia desnormalizada del correo) | Cualquier visitante ve `marta.ruiz@corriente.es` en cada artículo de Marta. Lo sufre la autora; en la interfaz nadie lo nota. | 1 | Sí | A ninguno (se arregla en el mismo transformer que C-03). |
| C-03 | **[R6 · Listar]** La misma respuesta expone el correo del autor otra vez, en `autor.email`. | API: `backend/start/routes.ts:119` (`select * from autores`) y `:168` | Igual que C-02, por una segunda vía: tapar solo `post.autor_email` no basta. | 1 | Sí | A ninguno (se arregla en el mismo transformer que C-02). |
| C-04 | **[Arquitectura · Crear]** La respuesta de crear un comentario devuelve la fila cruda sin transformer, con campos internos (`estado`, `post_id`), lo que incumple `CLAUDE.md:60` y `:62`. | API: `backend/start/routes.ts:329-333` (`select *` y `send({ comentario: nuevos[0] })`) | Nadie todavía: el `autor_email` que devuelve es el que quien comenta acaba de enviar (`routes.ts:306`), así que no ve el de nadie más. Por sí solo no incumple R6; solo lo incumple combinado con C-05. | 1 | Sí | Se arregla junto a C-05 (mismas líneas). |
| C-05 | **[R6 · Crear]** Después de insertar, el POST lee «el último comentario de la tabla» en lugar del que acaba de insertar. | API: `backend/start/routes.ts:330` (`select * from comentarios order by id desc limit 1`) | Si dos personas comentan a la vez, una recibe el comentario de la otra, con su correo incluido (R6). Con el tráfico de hoy, nadie todavía. | 1 | Sí | Se arregla junto a C-04. |
| C-06 | **[Arquitectura · Editar]** La respuesta de editar devuelve la fila cruda sin transformer, con campos internos (`estado`, `post_id`), lo que incumple `CLAUDE.md:60` y `:62`. | API: `backend/start/routes.ts:389-392` (`select *` y `send({ comentario: actualizados[0] })`) | Nadie todavía: para editar hay que enviar el correo exacto (`routes.ts:379`), así que el `autor_email` devuelto es el de quien edita. No incumple R6. | 1 | Sí | A ninguno |
| C-07 | **[Borrar]** Para borrar un comentario basta con conocer el correo de quien lo escribió, y C-01 se lo da a cualquiera. | API: `backend/start/routes.ts:350-353` (la «autorización» es comparar el `autor_email` que manda el cliente). Se combina con `routes.ts:153` | Cualquier visitante puede borrar cualquier comentario aprobado: lee el correo en `GET /posts/:slug` y manda `DELETE /comentarios/:id` con ese `autor_email`. Lo nota quien comentó, cuando su comentario desaparece, y el borrado es físico (`:359`). | 2 (DELETE y PATCH comparten el modelo de autorización; hace falta una identidad de verdad: token o enlace firmado) | Sí (el arreglo es código; lo ya borrado no vuelve) | C-01 lo mitiga, pero no lo arregla. Bloquea C-20: no se debe poner un botón de borrar con esta autorización. |
| C-08 | **[Borrar]** La clave de moderación está escrita en el código fuente. | API: `backend/start/routes.ts:25` (`'dev-secret-cambiar'`) y `:351` | Quien tenga acceso al repo puede borrar cualquier comentario con la cabecera `x-clave-moderacion`. Sale a la luz cuando alguien la use. | 2 (constante + `start/env.ts`/`.env`) | **No**: la clave ya está en el historial de git. Hay que rotarla, no basta con moverla. | A ninguno |
| C-09 | **[R5 · Listar]** El listado de posts cuenta como comentarios todos los que no están pendientes, en vez de solo los aprobados. | API: `backend/start/routes.ts:75-76` (`c.estado <> 'pendiente'`). Front: `frontend/src/componentes/ListaPosts.jsx:57`. BD: semilla `backend/database/seeders/blog_seeder.ts:510,522` | El lector ve «4 comentarios» en la tarjeta del post 1 y 2 al entrar. Un estado futuro como `rechazado` o `spam` también contaría. | 1 | Sí | Lo bloquea C-12 (antes hay que decidir qué es «aprobado»). |
| C-10 | **[R5 · Listar]** Los relacionados cuentan los comentarios con el mismo criterio `<> 'pendiente'`. | API: `backend/start/routes.ts:223`. Front: `frontend/src/componentes/PostsRelacionados.jsx:32` | El lector ve en la tarjeta de relacionados un número distinto del que encuentra al abrir el post. | 1 | Sí | Lo bloquea C-12 |
| C-11 | **[R5 · Listar]** El detalle filtra por `estado = 'aprobado'` exacto, así que oculta los comentarios aprobados que están guardados como `Aprobado` u `ok`. | API: `backend/start/routes.ts:153`. BD: semilla `blog_seeder.ts:510` (`'Aprobado'`), `:522` (`'ok'`), `:575` (`'Aprobado'`) | El post 6 tiene un comentario aprobado y no se ve: el listado dice 1 y el detalle 0. Lo notan Paula Merino, Rubén Lima y Silvia Peña, que no encuentran su comentario. | 1 | Sí | Lo bloquea C-12 |
| C-12 | **[R5 · BD]** `comentarios.estado` es texto libre, sin CHECK ni enum, y en él conviven `aprobado`, `Aprobado` y `ok`. | BD: `backend/database/migrations/1769000000005_create_comentarios_table.ts:18`. Datos: `blog_seeder.ts:510,522,575` | Nadie lo ve directamente, pero es la causa de que cada consulta entienda «aprobado» a su manera (C-09, C-10, C-11). | 2 (migración nueva con UPDATE de normalización + CHECK; semilla) | **No**: el UPDATE que normaliza `Aprobado`/`ok` a `aprobado` destruye información. El `down` no sabe qué variante tenía cada fila. | Desbloquea C-09, C-10 y C-11. |
| C-13 | **[R5 · Crear · Front]** Al enviar un comentario, que entra pendiente, el bloque de comentarios suma 1 al recuento. | Front: `frontend/src/componentes/Comentarios.jsx:39` (`setTotal(total + 1)`). La cabecera no se entera: `DetallePost.jsx:71` | Quien comenta ve «3 comentarios» con 2 visibles, y la cabecera del artículo sigue diciendo 2. | 1 | Sí | A ninguno |
| C-14 | **[R5 · Listar · Front]** Los dos recuentos del detalle cuentan la longitud del array, no los comentarios que se pintan: las respuestas sin padre visible cuentan pero no aparecen. | Front: `frontend/src/componentes/DetallePost.jsx:38` y `Comentarios.jsx:23` frente a `Comentarios.jsx:50` y `:63-64` | Si se borra el comentario 1, su respuesta (id 2) queda huérfana: la página dice «1 comentarios» y no enseña ninguno. Nadie todavía. | 2 (los dos componentes, o un único recuento compartido) | Sí | Lo provocan C-15 y C-16 |
| C-15 | **[Borrar]** Borrar un comentario que tiene respuestas deja esas respuestas apuntando a un padre que ya no existe. | API: `backend/start/routes.ts:359` (`delete` sin tratar las respuestas). BD: `1769000000005_create_comentarios_table.ts:20` (`padre_id` sin FK) | Las respuestas desaparecen de la interfaz pero siguen contando en el listado y en el detalle (C-14). Hoy pasaría al borrar el comentario 1. | 2 (servicio de borrado + FK/`ON DELETE`) | Sí, si solo se toca código. Si se añade la FK, la migración rehace la tabla en SQLite y no recupera los huérfanos que limpie. | Provoca C-14 |
| C-16 | **[Crear]** `padre_id` se guarda sin validar, así que se puede responder a un comentario pendiente, a uno de otro post o a uno que no existe. | API: `backend/start/routes.ts:308` y `:326`. BD: `1769000000005_create_comentarios_table.ts:20` (sin FK) | Una respuesta aprobada a un padre pendiente cuenta pero no se pinta. Nadie todavía: el front no envía `padre_id` (`Comentarios.jsx:33`). | 2 (validador + FK) | Sí (si se añade la FK, igual que en C-15) | Provoca C-14. Bloquea C-21: no se deben habilitar respuestas sin esta validación. |
| C-17 | **[R3 · Crear]** El POST localiza el post con `slug = ? limit 1`, sin orden y sin unicidad garantizada. | API: `backend/start/routes.ts:297`. BD: `1769000000004_create_posts_table.ts:20` (`slug` sin `unique`) | Si dos posts comparten slug, el comentario cae en uno indeterminado. Nadie todavía: los slugs de la semilla son únicos. | 1 (migración con índice `unique`) | Sí (el `down` quita el índice) | A ninguno |
| C-18 | **[Editar]** Al editar se puede dejar un cuerpo de 1 carácter, aunque al crear se exigen 5. | API: `backend/start/routes.ts:384` (`if (!cuerpo)`) frente a `:319` (`length < 5`) | Nadie todavía: no hay interfaz de edición. Por API, quien comenta puede vaciar su comentario. | 1 (validador compartido) | Sí | A ninguno |
| C-19 | **[Editar/Borrar]** Editar y borrar aplican reglas de autorización distintas para la misma persona. | API: `backend/start/routes.ts:379` (comparación exacta, sin clave de moderación) frente a `:351-353` (sin mayúsculas, con clave) | Quien escribió su correo con mayúsculas puede borrar pero no editar, y un moderador puede borrar pero no editar. Nadie todavía (no hay interfaz). | 2 | Sí | Se arregla junto a C-07 |
| C-20 | **[Editar/Borrar · Front]** El blog no tiene interfaz para editar ni para borrar comentarios: el único `fetch` que escribe es el POST. | Front: `frontend/src/componentes/Comentarios.jsx:31` (`method: 'POST'`, la única llamada que no es GET en todo `frontend/src`) | Quien comenta no puede corregir ni retirar su comentario desde la web. Lo nota cualquiera que se equivoque al escribir. | 1 (componente) | Sí | Lo bloquean C-07 y C-08 |
| C-21 | **[Crear · Front]** El formulario no puede enviar respuestas, aunque la vista sí pinta hilos. | Front: `frontend/src/componentes/Comentarios.jsx:33` (cuerpo sin `padre_id`) frente a `:63-64` | Los lectores no pueden responder. Las respuestas de la semilla (id 2) solo pueden haber entrado por fuera de la web. | 1 | Sí | Lo bloquea C-16 |
| C-22 | **[Crear]** La validación del POST responde con cuatro formas de error distintas, y un cuerpo demasiado corto devuelve `200`. | API: `backend/start/routes.ts:310-321` (`400 {error}`, `422 {message}`, `400 {errores}`, `200 {ok:false}`). Front: `frontend/src/componentes/Comentarios.jsx:44` lo tapa leyendo `error \|\| message \|\| motivo` | Un cliente que se fíe del código HTTP cree que el comentario corto se ha creado. El front de hoy se salva porque comprueba `d.comentario`. | 2 (validador VineJS + `Comentarios.jsx:44` si cambia la forma del error) | Sí | A ninguno (cambiarlo es otra tarea, con su propia caracterización previa) |
| C-23 | **[Tests]** La única prueba de comentarios solo comprueba `autor_nombre`. No cubre ninguna regla, ni la edición, ni el borrado, ni el listado. | `backend/tests/functional/comentarios.spec.ts:41-42` | Nadie, hasta que un refactor rompa cualquiera de las filas anteriores sin que falle nada. | 1 (suite de caracterización) | Sí | **Bloquea a todos los demás**: CLAUDE.md exige caracterizar en verde antes de corregir. |
| C-24 | **[R1 · Editar/Borrar]** PATCH y DELETE no comprueban si el post del comentario está publicado. | API: `backend/start/routes.ts:340` y `:367` (`select * from comentarios where id = ?`, sin cruzar con `posts.estado`) | El PATCH devolvería el comentario de un post en borrador. Nadie todavía: los borradores (posts 9 y 10) no tienen comentarios. | 2 | Sí | A ninguno |

---

## 3. Lo que sí se cumple (comprobado)

- **R2 en la API**: el detalle excluye los pendientes (`routes.ts:153`). Comprobado en vivo: el comentario 5 (pendiente) no sale.
- **R1 al crear y al listar**: no se puede comentar ni ver comentarios de un borrador (`routes.ts:297`, `:111`).
- **R2 en el front**: el front no añade a la lista el comentario recién enviado; solo muestra un aviso (`Comentarios.jsx:38`).

## 4. Orden recomendado para corregir

1. **C-23**: tests de caracterización de las cuatro acciones (desbloquea todo lo demás).
2. **C-08**: rotar la clave (no depende de nada y no se puede deshacer).
3. **C-01, C-02, C-03, C-05**: cierran R6 y le quitan a C-07 su vía de explotación. **C-04 y C-06** van en el mismo cambio: son los transformers de salida de crear y editar.
4. **C-07 y C-19**: autorización única y real para editar y borrar.
5. **C-12**, después **C-09, C-10, C-11, C-13, C-14**: un único criterio de «aprobado» en BD, API y front.
6. **C-15, C-16, C-24, C-17, C-18, C-22**.
7. **C-20, C-21**: interfaz de edición, borrado y respuestas, cuando la API sea segura.

# Las 3 líneas

Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo. Los dos números, tal cual salieron.

El agente encontró 24 hallazgos, y he podido comprobar 4 de ellos:
- Para los 4 hallazgos que he comprobado, le he pedido a Claude que me enseñe tanto el archivo como la línea en la que ese hallazgo se confirma.
- Para cada uno de ellos, he abierto el archivo para comprobar que efectivamente se cumplía lo que decía Claude. En los 4 casos, ha sido cierto.

2. El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué.
- Durante los 45 minutos que he tenido para realizar el ejercicio no he tenido tiempo de llegar a la parte de priorización, ya que me he quedado en la comprobación de los hallazgos.
- Revisando las prioridades que ha establecido Claude, las reordenaría de la siguiente manera (y por lo tanto las últimas 3 no sabría muy bien cómo priorizarlas):

```
1. **C-23**: tests de caracterización de las cuatro acciones (desbloquea todo lo demás).
2. **C-08**: rotar la clave (no depende de nada y no se puede deshacer).
3. **C-01, C-02, C-03, C-05**: cierran R6 y le quitan a C-07 su vía de explotación. **C-04 y C-06** van en el mismo cambio: son los transformers de salida de crear y editar.
4. **C-12**, después **C-09, C-10, C-11, C-13, C-14**: un único criterio de «aprobado» en BD, API y front.
**C-07 y C-19**: autorización única y real para editar y borrar.
**C-15, C-16, C-24, C-17, C-18, C-22**.
**C-20, C-21**: interfaz de edición, borrado y respuestas, cuando la API sea segura.
```

3. Algo que el agente afirmó y no pudiste verificar. No es lo que dijo mal: es lo que no pudiste comprobar ni a favor ni en contra.

- El hallazgo C-05 es el único de los que he revisado que no he podido comprobar.
- Generar una concurrencia en la que dos usuarios crean, simultáneamente, un comentario en el mismo post, y que uno pueda ver el email del otro al cargar el último comentario.