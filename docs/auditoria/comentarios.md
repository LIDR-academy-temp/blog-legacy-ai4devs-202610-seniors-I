# Auditoría priorizada — feature comentarios

Alcance: crear, editar, borrar y listar comentarios (API + tabla + web).

Criterio de orden: **1º precedencia** (lo que abarata o desbloquea a otro va antes); **2º daño ÷ radio** dentro del mismo nivel. Si un hallazgo junta el daño más alto con la precedencia más baja, se parte en **contención** (corta el daño hoy) y **arreglo de fondo** (espera su turno).

| # | Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios a tocar | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 1 | La suite solo caracteriza el POST feliz; no fija contrato ni reglas 2, 5 y 6. | `backend/tests/functional/comentarios.spec.ts:15-43` (aviso en `:8-10`). | Nadie todavía; sin esta red, cualquier cambio de email, recuento o status se puede romper en silencio. | 1: ese archivo de tests (matriz feliz / inválido / 404 / pendiente oculto / email ausente / recuento solo `aprobado`). | Sí. | Desbloquea **tocar con seguridad** los hallazgos 2, 3, 7 y 10 (no los arregla: el arreglo sigue siendo cada una de esas filas). |
| 2 | Contención: el detalle y las mutaciones devuelven `autor_email` vía `select *` / fila cruda. | `backend/start/routes.ts:153` (detalle), `:333` (POST), `:392` (PATCH). | Rompe la regla 6; lo nota quien abra el JSON o las DevTools. | 3: esas tres respuestas (columnas explícitas sin email, o un `omit` puntual). | Sí. | **Abarata** el hallazgo 4 (ya no se lee el correo en el JSON público); no lo arregla. El fondo del email es el 10; el de la autoría, el 12. |
| 3 | Contención: `num_comentarios` usa `estado <> 'pendiente'` mientras el detalle exige `estado = 'aprobado'`. | Recuentos en `routes.ts:75-76` y `:223`; filtro del detalle en `:153`. Semilla: `blog_seeder.ts:510` (`Aprobado`) y `:522` (`ok`). | En el post 1 el listado marca 4 y el detalle enseña 2; lo nota el lector. | 2: las dos queries de recuento (alinearlas a `estado = 'aprobado'`). | Sí. | **Corta el daño** del desajuste hoy; no cierra el 11 (fondo del `estado`). |
| 4 | Borrar un comentario basta con mandar el `autor_email` en el body. | `backend/start/routes.ts:350-356`. | Quien conozca el correo borra comentarios ajenos; lo nota el autor al verlos desaparecer. | 1: la autorización del DELETE. | Sí. | Lo abarata el hallazgo 2; desbloquea alinear el PATCH (hallazgo 8) sobre la misma regla. |
| 5 | La clave de moderación es la constante `dev-secret-cambiar`. | `backend/start/routes.ts:25`, usada en `:351`. | Quien lea el repo borra cualquier comentario con `x-clave-moderacion`; lo nota el equipo al exponer el secreto. | 2: constante → entorno, y el chequeo del DELETE. | Sí. | A ninguno. |
| 6 | Tras enviar, la sección hace `total + 1` aunque el comentario nace `pendiente` y no se lista. | `frontend/src/componentes/Comentarios.jsx:39`. | El `h2` miente al instante; lo nota quien acaba de comentar. | 1: esa línea (no incrementar, o refrescar solo aprobados). | Sí. | Abarata el hallazgo 9: sin este +1, los dos contadores dejan de divergir al enviar. |
| 7 | Un cuerpo de menos de 5 caracteres responde `200 { ok: false }` en vez de 4xx. | `backend/start/routes.ts:319-320`. | Quien solo mire el status HTTP cree que fue bien; la web actual lo mitiga mirando `d.comentario`. | 1: esa rama del POST (+ test que ya fijó el hallazgo 1). | Sí. | Lo desbloquea el hallazgo 1. |
| 8 | PATCH y DELETE no comparten autoría: PATCH compara el email tal cual y solo si sigue `pendiente`; DELETE normaliza y admite moderación. | `routes.ts:375-380` frente a `:348-356`. | El autor con otra capitalización (o el moderador) borra pero no edita; lo nota quien corrige un typo antes de la aprobación. | 2: DELETE y PATCH (una sola función de autorización). | Sí. | Lo desbloquea el hallazgo 4 (misma regla). |
| 9 | Cabecera del artículo y `h2` de la sección llevan contadores distintos. | `DetallePost.jsx:30/:38/:71` y `Comentarios.jsx:11/:54`. | Tras comentar (o al cargar), los dos números pueden no coincidir; lo nota el lector en la misma página. | 2: esos dos componentes (un solo origen de verdad). | Sí. | Lo abarata el hallazgo 6. |
| 10 | Arreglo de fondo del email: las respuestas siguen sin transformer; el contrato público no está en una capa. | No hay transformer de comentario; las tres salidas siguen en `routes.ts:153/:333/:392`. | Nadie todavía si la contención (2) ya cortó la filtración; sin esto, el próximo `select *` la reabre. | 1 capa nueva (transformer) + 3 call sites que la usen. | Sí el diff; no es un revert del esquema. | Espera a la contención 2; abarata no repetir el omit a mano. |
| 11 | Arreglo de fondo del `estado`: texto libre sin CHECK ni validador, con `Aprobado`/`ok` en semilla. | Migración `1769000000005_create_comentarios_table.ts:18` (`:3-6`); semilla `:510` y `:522`. | Vuelven a colarse estados que el detalle no lista; lo nota quien modera o re-siembra. | 3: migración/CHECK o enum, validador al escribir, limpieza de la semilla. | No: esquema y datos no vuelven con un revert de rutas. | Desbloquea que la contención 3 se mantenga sin reincidir; espera su turno tras cortar el daño. |
| 12 | Arreglo de fondo de la autoría: no hay sesión ni token de comentarista; email-en-body es el único factor. | DELETE `:350-356` y PATCH `:378-380`. | Nadie todavía si 2 y 4 ya redujeron la superficie; sin auth real, cualquiera que sepa un correo sigue pudiendo actuar. | 2: DELETE y PATCH (auth de verdad) + lo que consuma la web. | No del todo: implica contrato nuevo, no solo revert. | Espera a 2 y 4; es el fondo del daño más alto con la precedencia más baja. |

## Fuera de la tabla — no medido

Propuestas o daños condicionales: no entran a la prioridad hasta tener el número al lado.

| Propuesta | Qué habría que medir |
|---|---|
| Sustituir el `select … order by id desc limit 1` post-INSERT (`routes.ts:329-330`) por el id insertado. | ¿Hay envíos concurrentes reales? Tasa de `201` cuyo `comentario.cuerpo` / `autor_email` no coinciden con el body enviado. |
| Validar que `padre_id` exista y sea del mismo `post` (`routes.ts:308`, `:325-326`). | ¿Algún cliente manda `padre_id`? La web no lo envía (`Comentarios.jsx:33`). Conteo de filas cuyo `padre_id` apunta a otro post o a un id inexistente. |
| Añadir FK + índice a `comentarios.post_id` (migración `:14`). | ¿Existen huérfanos hoy? ¿Se borran posts fuera de esta API? Conteo `comentarios` sin `posts.id` coincidente. |
| Partir comentarios en capas (ruta → controlador → validador → servicio → repositorio → transformer). | Coste actual: líneas/archivos tocados por cada hallazgo de la tabla. Coste esperado tras el split. Sin ese antes/después, el «abarata todo» es una hipótesis. |

## Por qué el 1 desbloquea al 2, 3, 7 y 10 (siendo un test)

El test no corta el daño: **fija el contrato actual (bugs incluidos) para que el siguiente cambio se note**. Sin esa red, `npm test` sigue verde aunque 2, 3, 7 o 10 se «arreglen» a medias o se rompan al mover código. Hoy la suite solo afirma `201` + `autor_nombre` (`comentarios.spec.ts:41-42`); no mira email, recuento, status del cuerpo corto ni la forma completa de la respuesta.

| Hallazgo desbloqueado | Qué falta hoy en la suite | Qué desbloquea el test |
|---|---|---|
| **2** (contención email) | No hay `assert.notProperty(..., 'autor_email')` en detalle/POST/PATCH | Sin esa aserción negativa, quitar el email en un sitio y dejarlo en otro sigue en verde; con ella, la contención tiene criterio de hecho. |
| **3** (contención recuento) | Nadie afirma `num_comentarios` del listado frente a `comentarios.length` del detalle con filas `Aprobado`/`ok` | Sin ese contraste, alinear las dos queries a `estado = 'aprobado'` no tiene oráculo; el test lo da (hoy 4 vs 2 en post 1). |
| **7** (`200 {ok:false}`) | El cuerpo corto no está caracterizado | Primero se fija el bug (`200` + `ok:false`); eso desbloquea decidir el arreglo: al pasar a 4xx, el test rojo documenta el cambio de contrato a propósito (CLAUDE.md: caracterizar → luego corregir). |
| **10** (transformer) | No hay contrato completo del DTO de comentario | El refactor a transformer no puede mantener el verde «en cada paso» si nadie fijó qué campos salen; el test de caracterización es el candado del 10. |

## Por qué el 2 abarata al 4 (y no cierra el 10 ni el 12)

La contención no cambia el DELETE: **corta el canal gratis por el que el atacante obtiene el correo**. Hoy `GET /posts/:slug` hace `select` * de comentarios (`routes.ts:153`) y lo manda en `:171`; POST/PATCH devuelven la fila entera (`:333`, `:392`). Con ese JSON, el DELETE solo pide repetir `autor_email` en el body (`:350-356`). Sin el email en claro, el hallazgo 4 sigue existiendo, pero ya no se alimenta de la respuesta pública.

| Hallazgo                 | Qué demuestra el código hoy                                                                | Qué hace (o no hace) la contención 2                                                                                                                          |
| ------------------------ | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **4** (borrar con email) | El DELETE autentica comparando el email del body con `comentario.autor_email` (`:352-353`) | **Abarata**: quita la fuente pública del secreto. Quien solo miraba DevTools ya no tiene el correo; el arreglo del 4 sigue siendo endurecer esa autorización. |
| **10** (transformer)     | Las tres salidas siguen siendo fila cruda; no hay DTO                                      | **No lo cierra**: el omit puntual corta el daño hoy; el 10 espera para que el próximo `select` * no reabra el agujero.                                        |
| **12** (auth de fondo)   | No hay sesión ni token de comentarista; el factor sigue siendo conocer el correo           | **No lo cierra**: si el correo se conoce por otro medio, DELETE/PATCH siguen iguales; el 12 es el arreglo de autoría de verdad.                               |

## Por qué el 3 corta el daño hoy (y no cierra el 11)

La contención no limpia `estado`: **alinea el recuento con lo que el detalle ya lista**. Hoy el listado y los relacionados cuentan `estado <> 'pendiente'` (`routes.ts:75-76`, `:223`) y el detalle solo trae `estado = 'aprobado'` (`:153`). En el post 1 de la semilla eso son **4** frente a **2**: entran `Aprobado` (`blog_seeder.ts:510`) y `ok` (`:522`), que no son `pendiente` pero tampoco `aprobado`. Cambiar solo las dos queries de recuento a `estado = 'aprobado'` hace coincidir el número con lo visible; la regla 5 («solo los aprobados») queda cumplida en lectura sin tocar el esquema.

| Hallazgo | Qué demuestra el código / los datos hoy | Qué hace (o no hace) la contención 3 |
|---|---|---|
| **Daño visible (esta fila)** | Post 1: `<> pendiente` → 4; `= aprobado` → 2; estados reales: `aprobado`×2, `Aprobado`×1, `ok`×1, `pendiente`×1 | **Corta el daño**: listado y relacionados pasan a contar 2, igual que el detalle. El arreglo es esas 2 queries, no la migración. |
| **11** (fondo del `estado`) | `estado` es `string` libre (migración `:18`); la semilla y cualquier escritura pueden volver a meter `Aprobado`/`ok` | **No lo cierra**: sin CHECK/validador/limpieza, el desajuste puede reincidir en cuanto entre un estado nuevo que no sea exactamente `aprobado`. El 11 espera su turno para que la contención se mantenga. |

## Parte B — tres líneas (relleno humano)

1. Hallazgos del agente: **12 en tabla + 4 fuera**. Comprobados abriendo el archivo: 4
2. Prioridad dudosa:  El hallazgo 5— porque esta no afecta a Ninguno.
3. Afirmación no verificable: El hallazgo12 expone un problema de seguridad, no evidencie cuales eran los mecanismos de Autenticacion y Autorización .
