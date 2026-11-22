# Auditoría priorizada — feature «comentarios»

**Alcance:** crear, editar, borrar y listar comentarios en sus tres capas: API (`backend/start/routes.ts`),
base de datos (migración y semilla de `comentarios`) y web (`frontend/src/componentes/Comentarios.jsx`
y el recuento de `DetallePost.jsx`). Las demás features quedan fuera.

**Método:** solo he leído el código. No he arrancado la aplicación porque este worktree no tiene
`node_modules` ni `tmp/db.sqlite3`. Los recuentos por post salen de contar a mano las filas de la
semilla (`blog_seeder.ts:479-580`), no de ejecutar nada.

**Orden:** primero la precedencia (lo que abarata o desbloquea otro hallazgo va antes) y,
dentro de cada nivel, el daño dividido por el radio. El daño es una escala: alto = 3,
medio = 2, bajo = 1 y «nadie todavía» = 0. El radio es el número de sitios que hay que tocar.
Los dos hallazgos que más daño hacen y que dependen de otros se parten en contención (C-1, C-2)
y arreglo de fondo (#7, #6). El número de cada fila es un identificador fijo, no su puesto: el puesto lo da el orden de las filas.

---

## Nivel 0 — contenciones y lo que desbloquea todo lo demás

| # | Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| C-1 | **Contención:** el detalle público de un post devuelve el `autor_email` de cada comentarista. | `backend/start/routes.ts:153` → `"select * from comentarios where post_id = ? and estado = 'aprobado' …"`; sale tal cual en `routes.ts:171` (`comentarios: comentarios`). | Cualquier visitante o *scraper* se lleva el correo de quien comenta (incumple la regla 6). Además es la llave del #9: con ese correo se borran comentarios ajenos. | 1 (`routes.ts:153`: cambiar `*` por las columnas que pinta la web, que son `id, autor_nombre, cuerpo, creado_en, padre_id`) | Sí el código. **No** la exposición: los correos que ya se han servido siguen filtrados. | Desbloquea el #9 (corta de dónde salen los correos). Su arreglo de fondo es el #7. |
| C-2 | **Contención:** la clave de moderación está escrita en el código fuente. | `backend/start/routes.ts:25` → `const SECRETO_SESION = 'dev-secret-cambiar'`, que se compara en `routes.ts:351`. | Cualquiera que lea el repositorio (público y forkeado) manda `x-clave-moderacion: dev-secret-cambiar` y borra cualquier comentario. Lo notan los comentaristas y el moderador cuando faltan comentarios. | 3 (`routes.ts:25`, `backend/start/env.ts` para declararla y `backend/.env.example`) | **No**: el valor ya está en el historial de git. Hay que rotarlo, y un revert volvería a publicar un secreto. | Su arreglo de fondo es el #6. |
| 3 | La feature tiene un único test, que solo cubre el camino feliz del POST y solo afirma un campo. | `backend/tests/functional/comentarios.spec.ts:15` (`'deja un comentario en un post publicado'`) y `:42` (solo `autor_nombre`). | Nadie todavía. Pero cualquier refactor o arreglo de esta tabla puede cambiar el contrato sin que nada falle. | 1 archivo de spec, con 3 grupos de caracterización (POST, PATCH y DELETE) más el bloque `comentarios` del GET de detalle | Sí (solo añade tests). | Desbloquea los #4, #5, #8, #10 y #11, todo lo que cambia el contrato o el comportamiento. |

## Nivel 1 — los desbloquea el nivel 0 y desbloquean el resto

| # | Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 4 | `comentarios.estado` es texto libre, sin CHECK ni enum, y en la semilla conviven `'aprobado'`, `'Aprobado'` y `'ok'`. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:18` → `table.string('estado').notNullable().defaultTo('pendiente')`. Datos: `blog_seeder.ts:510` y `:575` (`'Aprobado'`) y `:522` (`'ok'`). | Los comentarios 3, 4 y 9, aprobados en la práctica, no salen en el detalle porque `routes.ts:153` compara exacto con `'aprobado'`. Lo notan quienes los escribieron y los lectores de los posts 1 y 6. | 4 (1 migración nueva con UPDATE + CHECK, y las 3 líneas de la semilla) | **No**: el UPDATE que normaliza borra la diferencia entre `'ok'` y `'Aprobado'` si no se guarda antes una copia. | Lo desbloquea el #3. Abarata el #12, porque decide qué cuenta como «aprobado». |
| 5 | Toda la feature vive en las rutas: SQL, validación, autorización y forma de la respuesta, sin controlador, validador, servicio, repositorio ni transformer. | `backend/start/routes.ts:295` (POST), `:339` (DELETE) y `:366` (PATCH): son *closures* con `db.rawQuery`. | Nadie todavía. Cada regla está copiada a mano y ya ha divergido (#10). | 10 (3 rutas que se reducen a `[Controlador, 'metodo']`, 6 archivos nuevos y el alias `#repositories/*` en `backend/package.json`) | Sí, si la caracterización del #3 sigue en verde. | Lo desbloquea el #3. Desbloquea los #6, #7, #8 y #13 (validador, transformer y repositorio). |

## Nivel 2 — los desbloquea el nivel 1 y a su vez desbloquean otro hallazgo

Orden: 6 → 9 → 8 → 7. Por daño ÷ radio, #9 (3÷2 = 1,5) iría delante de #6 (2÷3 ≈ 0,67), pero #6 desbloquea a #9, y la precedencia manda también dentro del nivel. Después van #8 (2÷4 = 0,5) y #7 (1÷3 ≈ 0,33).

| # | Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 6 | **Fondo de C-2:** la moderación es un secreto compartido en una cabecera, no un usuario autenticado. | `backend/start/routes.ts:349-351` → `request.header('x-clave-moderacion')` … `clave === SECRETO_SESION`. Ya existe `middleware.auth()` en `routes.ts:416`, pero no se usa aquí. | Sin C-2, cualquiera modera. Con C-2, quien filtre la clave modera, y no queda registro de quién borró qué. | 3 (`routes.ts:349-351`, una migración nueva con un rol en `users`, `backend/app/models/user.ts`) | Sí el código. La migración se deshace con su `down`, pero pierde los roles asignados. | Lo desbloquea el #5. Desbloquea los #9 y #10. |
| 9 | Para borrar o editar como autor basta con mandar el email del comentarista en el cuerpo de la petición. | `backend/start/routes.ts:350-353` → `request.input('autor_email', '')` … `=== String(email)…`, y `routes.ts:378-379` en el PATCH. | Mientras C-1 no esté hecho, cualquier lector copia el correo del detalle y borra el comentario de otro. Después de C-1, basta con conocer el correo de la víctima. Lo nota el comentarista cuando su comentario desaparece. | 2 (`routes.ts:350` y `:378`) | Sí. | Lo desbloquean C-1 (contiene el daño) y el #6 (prueba de identidad real). Desbloquea el #10. |
| 8 | La validación del POST devuelve cuatro formas y tres códigos distintos, y el cuerpo corto responde 200. | `backend/start/routes.ts:310-321`: `400 {error}` (`:311`), `422 {message}` (`:314`), `400 {errores:[…]}` (`:317`) y `200 {ok:false, motivo}` (`:320`). | La web, con un email sin `@`, enseña el genérico «No se ha podido enviar.», porque `Comentarios.jsx:44` no lee `errores`. Lo nota quien comenta. Un cliente de la API no puede tratar los errores de forma uniforme. | 4 (`routes.ts:311`, `:314`, `:317` y `:320`), que pasan a un validador VineJS | Sí el código. Es un cambio de contrato para los clientes. | Lo desbloquean los #3 y #5. Desbloquea el #11 (el validador compartido trae el mínimo al PATCH). |
| 7 | **Fondo de C-1:** ninguna respuesta de comentarios pasa por un transformer, se devuelve la fila cruda. | `backend/start/routes.ts:171` (detalle), `:333` (`{ comentario: nuevos[0] }`) y `:392` (`{ comentario: actualizados[0] }`). | Contenido por C-1 en el detalle. En el POST y el PATCH se devuelven todas las columnas (`estado`, `post_id`, el correo). | 3 | Sí. | Lo desbloquea el #5. Desbloquea el #13 (deja de importar qué fila se lee si solo sale lo que permite el DTO). |

## Nivel 3 — hojas (no desbloquean nada), por daño ÷ radio

Orden: #14 (2÷1 = 2), #15 (2÷1 = 2, aunque solo cuando se borra un padre), #12 (2÷2 = 1), #10 (1÷2 = 0,5), #17 (1÷6 ≈ 0,17). Los de daño 0 empatan a cero y se desempatan con un criterio añadido: primero #13, porque cuando ocurra filtra un correo; después la cadena #16 → #18; al final #11 y #19.

| # | Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 14 | La web suma 1 al recuento al enviar, aunque el comentario queda pendiente y no se muestra. | `frontend/src/componentes/Comentarios.jsx:39` → `setTotal(total + 1)`. | Quien comenta ve «3 comentarios» con 2 visibles, y la cabecera (`DetallePost.jsx:71`) sigue diciendo 2. Lo nota cada comentarista. | 1 | Sí. | A ninguno. Su fondo es el #17. |
| 15 | Borrar un comentario padre deja huérfanas sus respuestas, que desaparecen de la web. | `backend/start/routes.ts:359` → `'delete from comentarios where id = ?'`, sin cascada ni FK (`migración …005:20`, `padre_id` sin `references`). La web solo pinta respuestas colgando de una raíz viva (`Comentarios.jsx:50` y `:64`). | Si se borra el comentario 1, el 2 (respuesta aprobada) deja de verse aunque sigue en la base. Lo nota quien escribió la respuesta. | 1 (`routes.ts:359`) | Depende del arreglo: si es **borrado en cascada, no**, porque el revert no devuelve las filas borradas. Si es «ocultar como `[borrado]`», sí. | A ninguno. |
| 12 | El recuento del listado y de relacionados cuenta todo lo que no es `pendiente`, en vez de solo lo aprobado. | `backend/start/routes.ts:75-76` → `c.estado <> 'pendiente'`, y `:223` (mismo filtro en relacionados), frente a `:153`, que compara exacto con `'aprobado'`. | Según la semilla, el post 1 (`diez-tendencias-…`) dice «4 comentarios» en el listado y enseña 2 en el detalle. El post 6 (`que-mide-de-verdad-…`) dice 1 y enseña 0. Lo nota cualquier lector (incumple la regla 5). | 2 (`routes.ts:76` y `:223`) | Sí. | Lo abarata el #4 (sin normalizar, el arreglo hace desaparecer del recuento los comentarios 3, 4 y 9). A ninguno. |
| 10 | PATCH y DELETE aplican reglas de autoría distintas: el PATCH distingue mayúsculas en el email, no admite moderador y solo edita pendientes. | `backend/start/routes.ts:379` → `comentario.autor_email !== email \|\| comentario.estado !== 'pendiente'`, frente a `:353` (`toLowerCase().trim()`) y `:355` (moderador). | Quien escribió su correo con otras mayúsculas puede borrar su comentario pero no editarlo, y el moderador no puede corregir uno. Hoy lo notaría solo un cliente de la API, porque la web no tiene editar ni borrar. | 2 (`routes.ts:353` y `:379`), que pasan a una sola regla en el servicio | Sí. | Lo desbloquean los #6 y #9. A ninguno. |
| 17 | Dos componentes piden el mismo detalle y guardan dos recuentos independientes de comentarios. | `frontend/src/componentes/DetallePost.jsx:30`, `:34` y `:38`, frente a `Comentarios.jsx:11`, `:19` y `:23`. | Dos peticiones a `/posts/:slug` por cada visita, y dos números que se desincronizan (#14). Lo nota el lector cuando comenta. | 6 (las seis líneas citadas: subir el estado a `DetallePost` y pasar `comentarios` como prop) | Sí. | Resuelve de fondo el #14. A ninguno más. |
| 13 | El POST devuelve «el último comentario de toda la tabla», no el que acaba de insertar. | `backend/start/routes.ts:329-331` → `'select * from comentarios order by id desc limit 1'`. | Si dos POST se cruzan entre el `await` del insert y el select, uno recibe el comentario del otro, con su correo. Nadie todavía (no medido, el tráfico es bajo). | 1 (usar el id que devuelve el insert) | Sí. | Lo abaratan los #5 (repositorio) y #7. A ninguno. |
| 16 | `padre_id` se guarda sin validar: puede apuntar a un comentario que no existe o a uno de otro post. | `backend/start/routes.ts:308` → `request.input('padre_id', null)`, que se inserta en `:326`. No hay FK (`migración …005:20`). | La respuesta queda invisible para siempre en la web. Nadie todavía: la web no envía `padre_id` (`Comentarios.jsx:33`), solo un cliente directo de la API. | 2 (regla en el validador y FK en una migración nueva) | Sí (la migración tiene `down`). | Lo abarata el #8 (validador). Decide la profundidad de respuestas que necesita el #18. |
| 18 | La web solo pinta un nivel de respuestas, así que una respuesta a una respuesta no se ve nunca. | `frontend/src/componentes/Comentarios.jsx:63-64` → `comentarios.filter((r) => r.padre_id === c.id)`, solo sobre las raíces. | Nadie todavía: la semilla no tiene ninguna y la web no permite responder. | 1 | Sí. | Depende del #16 (si la API limita la profundidad a 1, no hay que arreglar nada aquí). |
| 11 | El PATCH no aplica el mínimo de 5 caracteres que sí exige el POST. | `backend/start/routes.ts:384` → `if (!cuerpo)`, frente a `:319` → `String(cuerpo).length < 5`. | Un comentario pendiente se puede dejar en 1 carácter editándolo. Nadie todavía (la web no edita). | 1 | Sí. | Lo desbloquea el #8. A ninguno. |
| 19 | Los comentarios se pintan con `key={index}`. | `frontend/src/componentes/Comentarios.jsx:57` y `:66`. | Nadie todavía. Cuando la lista cambie en vivo (borrar o insertar), React puede reutilizar el nodo equivocado. | 2 | Sí. | A ninguno. |

---

## Fuera de la tabla — propuestas que nadie ha medido todavía

| Propuesta | Qué habría que medir antes |
|---|---|
| Índice en `comentarios.post_id` (`migración …005:14`, sin índice ni FK). | `EXPLAIN QUERY PLAN` y el tiempo de la subconsulta de `routes.ts:75` con un volumen realista de comentarios. Hoy hay 9 filas. |
| Límite de peticiones o antispam en `POST /posts/:slug/comentarios`. | Comentarios pendientes por IP y día, y cuántos acaban como spam en moderación. |
| Botones de editar y borrar en la web (hoy el PATCH y el DELETE solo existen en la API). | Cuántas peticiones PATCH/DELETE llegan hoy y si alguien lo ha pedido. |
| Paginar los comentarios del detalle (`routes.ts:152-155` los trae todos). | El máximo y el percentil 95 de comentarios aprobados por post. |

---

## Parte B — las tres líneas

1. **Hallazgos que devolvió el agente:** 21 filas (19 numeradas + las contenciones C-1 y C-2). **Comprobados por mí abriendo el archivo:** 21.
2. **El hallazgo del que no supe decidir la prioridad:** el #3 (falta de tests), porque su lugar en el nivel 0 sale de la norma de TDD y caracterización de `CLAUDE.md`, no de un bloqueo técnico: técnicamente solo bloquea el refactor #5.
3. **Algo que el agente afirmó y no pude verificar:** que dos POST simultáneos puedan cruzarse entre el insert y el `select … order by id desc limit 1` (#13), y que uno reciba el comentario del otro con su correo. El código lo permite, pero ni el agente ni yo lo reprodujimos con peticiones concurrentes, así que no puedo confirmarlo ni descartarlo.
