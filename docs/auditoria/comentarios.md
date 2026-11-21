# Auditoría priorizada — feature de comentarios

Alcance: crear, editar, borrar y listar comentarios en sus tres capas (BD, API, web).
Contraste: reglas de negocio de `README.md:59-65` y reglas de `CLAUDE.md:59-65`.
Orden: primero la dependencia (un hallazgo va después del que lo desbloquea) y, dentro del mismo
nivel, daño ÷ radio de mayor a menor. Daño: 3 = datos personales o irreversible, 2 = rompe una regla
con efecto visible y reversible, 1 = solo rompe `CLAUDE.md`. Radio = puntos de afectación (casilla 4).

| Nivel | Daño ÷ radio |
|---|---|
| R3 · sin dependencia | 3 ÷ 2 = 1,50 |
| H2b · sin dependencia | 2 ÷ 3 = 0,67 |
| H1 · sin dependencia | 3 ÷ 6 = 0,50 |
| H2 · sin dependencia | 2 ÷ 8 = 0,25 |
| R4 · depende de H1 | 3 ÷ 6 = 0,50 |
| R5 · depende de R4 | 1 ÷ 2 = 0,50 |

Hallazgo = comprobado o visto en el código; R = riesgo (efecto deducido, no ejecutado).

| # | 1. Qué es | 2. Dónde está (línea) | 3. Qué se rompe y quién lo nota | 4. Sitios a tocar (puntos de afectación) | 5. ¿Se deshace con un revert? | 6. ¿Bloquea o desbloquea a otro? |
|---|---|---|---|---|---|---|
| R3 | Crear un comentario responde con la última fila de toda la tabla, no con la creada. | `backend/start/routes.ts:329-331` (`order by id desc limit 1`, sin `where`) | Con peticiones simultáneas devolvería el comentario de otra persona, con su correo. Nadie todavía. | 1 — `routes.ts:329-331` (2 puntos) | Sí. | |
| H2b | Al enviar un comentario, el contador suma uno aunque nace `pendiente`. | `frontend/src/componentes/Comentarios.jsx:39` (`setTotal(total + 1)`); estado de creación en `routes.ts:326` | Rompe la regla 5 (`README.md:64`). Lo ve quien comenta, en `Comentarios.jsx:54`, hasta recargar; la cabecera (`DetallePost.jsx:30`) no cambia y los dos números dejan de coincidir. Comprobado en la web: al enviar, el comentario queda pendiente pero el contador sube en 1, y al recargar vuelve a su valor. | 1 — `Comentarios.jsx:39` (3 puntos) | Sí. | A ninguno. |
| H1 | El detalle de un post devuelve el correo de cada comentarista. | `backend/start/routes.ts:153` (`select * from comentarios`) → `:171` | Rompe la regla 6 (`README.md:65`, `CLAUDE.md:61`). Lo ve cualquiera que llame a la API o mire la pestaña Red; la web no lo pinta (`Comentarios.jsx:59,68`). Comprobado con curl. | 1 — `routes.ts:153` (6 puntos) | Sí. | Desbloquea a R4: sin los correos de `routes.ts:171`, el borrado de `routes.ts:350-353` deja de estar al alcance. |
| H2 | El recuento cuenta todo lo que no es `pendiente`, pero el detalle solo muestra `aprobado`. | `backend/start/routes.ts:75-76` y `:223` (`<> 'pendiente'`) frente a `:153` (`= 'aprobado'`); `migrations/1769000000005_create_comentarios_table.ts:18` (`estado` texto libre) | Rompe la regla 5 (`README.md:64`). Lo ve quien lee el blog: el número de `ListaPosts.jsx:57` y `PostsRelacionados.jsx:32` no coincide con el del detalle (`DetallePost.jsx:38`, `Comentarios.jsx:23`). Datos: `blog_seeder.ts:510,522,575` (`'Aprobado'`, `'ok'`). Comprobado con GET: `diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna` da 4 en el listado frente a 2 en el detalle. | 4 — BD 2 (`migración:18` + normalizar los valores de `blog_seeder.ts:510,522,575`) · API 2 (`routes.ts:76`, `:223`) (8 puntos) | La API, sí. La BD, no: normalizar `'Aprobado'`/`'ok'` sobrescribe el valor original. | A ninguno. |
| R4 | Para borrar basta con mandar el correo del autor o una clave escrita en el código. | `backend/start/routes.ts:25` (`SECRETO_SESION`), `:350-353`, `:359` (`delete` físico) | Cualquiera con un correo de H1 o con la clave borra comentarios ajenos, y sus respuestas dejan de pintarse (`migración:20` sin clave foránea; `Comentarios.jsx:50,63-64`). Nadie todavía: no hay `DELETE` en `frontend/src`. | 3 — `routes.ts:25`, `:350-353`, `:359` (6 puntos) | El código, sí. La clave, no: está en el historial de git desde `e0e0943`. | Lo desbloquea H1. Bloquea a R5. |
| R5 | La regla para editar está reescrita y no coincide con la de borrar. | `backend/start/routes.ts:379` frente a `:351-353` | Quien escribe su correo con otras mayúsculas no puede editar, y moderación nunca puede. Nadie todavía: no hay `PATCH` en `frontend/src`. | 1 — `routes.ts:379` (2 puntos) | Sí. | Bloqueado por R4: la regla común se define al arreglar R4. |

## Parte B

1. Cuántos hallazgos devolvió el agente, y cuántos comprobé yo abriendo el archivo.
   6 devueltos, 4 comprobados.

2. El hallazgo del que no supe decidir la prioridad, y por qué.
   H2b: sumar uno al contador al enviar un comentario parece intrascendente (se corrige al recargar)
   e incluso roza la duda de si está hecho a propósito, como una actualización optimista
   (`Comentarios.jsx:8-10` lo describe sin marcarlo como error).

3. Algo que el agente afirmó y no pude verificar.
   R4: que con los correos expuestos por H1 se puedan borrar comentarios ajenos
   (`routes.ts:350-353`, `:359`). Comprobarlo exige lanzar un DELETE, fuera del alcance de una
   auditoría de solo lectura.
