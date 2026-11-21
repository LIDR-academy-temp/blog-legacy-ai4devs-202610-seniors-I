# Auditoría priorizada — feature de comentarios

Alcance: crear, editar, borrar y listar comentarios en API (`backend/start/routes.ts`), BD (migración `1769000000005`, semilla) y web (`Comentarios.jsx`, `DetallePost.jsx`, `ListaPosts.jsx`, `PostsRelacionados.jsx`). Nada más.
Línea base `make test`: 1 test, 1 pasa. Comprobaciones con la API viva en :3401 y la semilla (re-sembrada después).

Orden: precedencia primero (lo que abarata o desbloquea a otro va antes); dentro de cada nivel, daño / radio.

| # | Qué es | Dónde (línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios | ¿Revert? | Bloquea / desbloquea |
|---|---|---|---|---|---|---|
| 1 | Solo hay un test (camino feliz del POST); nada fija el comportamiento actual | `tests/functional/comentarios.spec.ts:15` `test('deja un comentario en un post publicado'` | Nadie todavía; cualquier arreglo de abajo puede romper algo sin que se note | 1 archivo (crear tests de caracterización) | Sí | Desbloquea a todos |
| 2a | **Contención:** el detalle público devuelve el email de cada comentarista | `routes.ts:153` `"select * from comentarios where post_id = ? ...` | Cualquier visitante lee los correos (comprobado en `GET /posts/diez-tendencias…`); viola la regla 6 | 1 | No: los correos ya expuestos no se des-exponen | Corta hoy la recolección que explota 3 |
| 3 | Borrar un comentario solo exige conocer el email del autor (sin mayúsculas ni espacios) | `routes.ts:352-353` `String(comentario.autor_email).toLowerCase().trim() === ...` | Cualquiera borra comentarios ajenos con el email leído en 2a (comprobado: `DELETE /comentarios/2` con `" MARTA.RUIZ@corriente.es"` → 200) | 1 ruta + diseño de identidad (no contado) | No: el borrado es `delete` físico, el comentario se pierde | Bloqueado por decidir el modelo de autenticación; bloquea a 9 |
| 4 | La clave de moderación está escrita en el código de un repo público | `routes.ts:25` `const SECRETO_SESION = 'dev-secret-cambiar'` y `:351` | Cualquiera que lea el repo borra cualquier comentario (comprobado: `DELETE /comentarios/1` con la cabecera → 200) | 2 (+ `.env.example`) | No: la clave queda en el historial de git; hay que rotarla | A ninguno |
| 5 | `estado` es texto libre y conviven `aprobado`, `Aprobado` y `ok` | migración `:18` `table.string('estado')` sin CHECK; semilla: 4 `aprobado`, 2 `Aprobado`, 1 `ok` | Comentarios moderados como aprobados no se ven en el detalle (`routes.ts:153` compara exacto). Lo nota el comentarista | 1 migración de datos + 1 CHECK | Datos: no sin copia previa | Desbloquea 6 |
| 6 | El recuento cuenta "no pendiente" en vez de "aprobado" | `routes.ts:75-76` y `:223` `estado <> 'pendiente'` | El listado dice 4 comentarios del post 1 y el detalle enseña 2 (comprobado). Lo nota el lector; viola la regla 5 | 2 | Sí | Bloqueado por 5; desbloquea 10 |
| 7 | `padre_id` no se valida y borrar un padre deja hijos huérfanos | `routes.ts:308` `request.input('padre_id', null)`, `:359` `delete ... where id = ?` | Se aceptó `padre_id: 99999` (201). Tras borrar el 1, la respuesta 2 sigue contando pero `Comentarios.jsx:50` ya no la pinta | 2 back + 1 front | Sí el código; los huérfanos ya creados, no | A ninguno |
| 8 | Cuatro formatos de error distintos, y el cuerpo corto responde 200 | `routes.ts:310-321` (`400 error`, `422 message`, `400 errores`, `200 {ok:false}`) | El front los parchea leyendo tres claves (`Comentarios.jsx:44`); un cliente nuevo trata el 200 como éxito | 4 back + 1 front | Sí | Hay que tocarlo junto con el front |
| 9 | PATCH y DELETE aplican reglas de autoría distintas | `routes.ts:379` `comentario.autor_email !== email` frente a `:352-353` | Con el mismo email puedes borrar pero no editar si cambian las mayúsculas; el moderador no puede editar | 2 | Sí | Bloqueado por 3 (misma regla de identidad) |
| 10 | Dos recuentos en el front que se separan al comentar | `DetallePost.jsx:30` y `Comentarios.jsx:11`, `:39` `setTotal(total + 1)` | La cabecera y la sección enseñan números distintos tras enviar; además se suma un pendiente que no se ve | 2 | Sí | Bloqueado por 6 |
| 11 | El POST devuelve "el último comentario por id", no el insertado | `routes.ts:329-330` `select * from comentarios order by id desc limit 1` | Con dos POST simultáneos, uno podría recibir el comentario (y el email) del otro | 1 | Sí | |

### Propuestas sin medir (fuera de la tabla)

- **Índice/FK en `comentarios.post_id`** (migración `:14`, sin índice). Medir: tiempo de `GET /posts` con N comentarios sembrados (por ejemplo 10k), antes y después.
- **Límite de envío / antispam en el POST.** Medir: cuántos comentarios basura llegan hoy. No hay datos de producción en el repo.

### Fuera de alcance (visto, no auditado)

- `GET /posts/:slug` también expone `post.autor_email` (`routes.ts:111`, `select *`). Pertenece a la feature de posts.

## Las tres líneas

1. Hallazgos devueltos por el agente: 11 (+2 propuestas sin medir). Comprobados por mí abriendo el archivo: 0. El agente comprobó 7 contra la API viva (2a, 3, 4, 5, 6, 7, 8); yo no abrí ninguno.
2. Hallazgo cuya prioridad no supe decidir: la clave de moderación en el código (4) frente al borrado por email (3). La 4 es la más grave y la más barata de cortar, pero no desbloquea nada; la 3 hace el mismo daño con un dato que ya filtra la API, y su arreglo de fondo espera a una decisión de autenticación que no es mía.
3. Algo que el agente afirmó y no pude verificar: que dos POST simultáneos devuelven el comentario (y el email) del otro (fila 11). No se reprodujo la concurrencia, así que no sé si pasa con SQLite en un solo proceso.
