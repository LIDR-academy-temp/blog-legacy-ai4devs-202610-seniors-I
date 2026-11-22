# Auditoría AI For Legacy — Comentarios

- **Fecha:** 2026-11-22
- **Rama:** `auditoria-pa`
- **Estado:** entregable del ejercicio. No incluye cambios de código.

## 1. Alcance

Solo la funcionalidad de comentarios: crear, editar, eliminar y listar, en frontend, API y persistencia SQLite.

| Capa | Archivos revisados |
|---|---|
| API | `backend/start/routes.ts` (rutas de comentarios y consultas que cuentan o listan comentarios) |
| Persistencia | `backend/database/migrations/1769000000005_create_comentarios_table.ts`, `backend/database/seeders/blog_seeder.ts` |
| Frontend | `frontend/src/componentes/Comentarios.jsx`, `frontend/src/componentes/DetallePost.jsx` |
| Tests | `backend/tests/functional/comentarios.spec.ts` (un solo test, que solo afirma `autor_nombre`) |

**Fuera de alcance:**
- El resto del blog.
- `posts.autor_email` y `autores.email`, que también salen por `select *` en `GET /posts/:slug` (`routes.ts:110-111`, `119`). Tienen la misma causa que H1, pero no son de comentarios.

**Hecho:** el frontend no tiene pantalla para editar ni borrar. La única escritura es el `POST` de `Comentarios.jsx:30-34`. Editar y borrar existen solo en la API.

**Aclaración:** la constante `SECRETO_SESION` (`routes.ts:25`) solo se usa en `routes.ts:351`, como clave de moderación. El cifrado de la app usa `APP_KEY`, que se lee del entorno (`start/env.ts:22`, `config/encryption.ts:16`). El comentario de `routes.ts:21`, que dice que esa clave firma las sesiones, no se corresponde con el código.

## 2. Hallazgos verificados

| Id | Hallazgo | Cómo se verificó |
|---|---|---|
| H1 | Se expone `autor_email` en respuestas públicas | Código y HTTP |
| H2 | Para borrar o editar basta con el correo; las dos reglas son distintas; la clave de moderación está fija en el código | Código |
| H3 | Un post cuenta 4 comentarios en el listado y devuelve 2 en el detalle | Código y HTTP |
| H4 | Una validación fallida devuelve HTTP 200 (`{ok:false}`); el resto de errores usa otros códigos y formatos | Código y HTTP |
| H5 | Sin claves foráneas y sin validar `padre_id` | Código. La consulta SQLite encontró 0 comentarios huérfanos |
| H6 | Tras crear, se devuelve el último comentario de toda la tabla | Código. La condición de carrera **no** se ha reproducido |

### Relación entre H1 y H2

H1 facilita conseguir el correo de quien comenta, pero no es la causa de H2. La autorización por correo falla igual con cualquier correo conocido por otra vía: no se verifica y no es un secreto.

- Ocultar `autor_email` (H1-C) reduce una vía de obtención de correos, pero **no corrige** la autorización.
- Ninguna de las dos depende técnicamente de la otra.

## 3. Tabla de trabajos

**Orden:**
- Primero por precedencia: lo que desbloquea a otros va antes y lo bloqueado por una decisión, al final.
- Dentro del mismo nivel, por daño en relación con el radio.

No se usan puntuaciones ni severidades. **C** significa contención y **D**, arreglo definitivo. Las filas marcadas *(nivel N)* abren cada nivel.

| Qué es | Dónde está | Qué se rompe y quién lo nota | Sitios que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea? |
|---|---|---|---|---|---|
| **H1-C** *(nivel 1: contenciones, en paralelo)*. Dejar de devolver `autor_email` en las respuestas de comentarios | `routes.ts:152-153` y `171` (detalle); `routes.ts:329-333` (respuesta del POST) | Cualquier visitante obtiene el correo de quien comenta. Quien comenta no lo nota | 2: `routes.ts` y `comentarios.spec.ts` (aserción negativa). El frontend no lee `autor_email` (`Comentarios.jsx:59-70`) | **El arreglo, sí:** solo es código. **La exposición ya ocurrida, no:** ningún revert recupera los correos ya servidos | No tiene dependencias técnicas. Reduce una vía de obtener correos, pero **no** corrige H2 |
| **H2-C1**. Restringir temporalmente el borrado y la edición autorizados solo por un correo no verificado | `routes.ts:350-353` (DELETE por correo); `routes.ts:378-379` (PATCH por correo) | Hoy: cualquiera que conozca un correo puede borrar los comentarios de esa persona; el afectado lo descubre cuando ya han desaparecido. Con la medida: quien comenta pierde la posibilidad legítima de editar o borrar lo suyo | 2: `routes.ts` y `comentarios.spec.ts`. Sin cambios en el frontend: no hay pantalla para editar ni borrar | **El código, sí. Los comentarios borrados antes de la medida, no:** el borrado es definitivo (`routes.ts:359`) y solo se recuperan con una copia de seguridad, que no está verificada | **Requiere una decisión funcional:** puede bloquear operaciones legítimas. No depende técnicamente de H1-C ni de H2-C2. H2-D la sustituye |
| **H2-C2**. Sustituir la clave de moderación fija por una nueva leída del entorno | `routes.ts:25` y `351` | Quien conozca la clave actual puede borrar cualquier comentario. La clave está en git desde `e0e0943` | 5: `routes.ts`, `start/env.ts`, `.env.example`, `.env.test` y `comentarios.spec.ts` | **El código, sí, pero revertirlo devuelve la clave comprometida.** La clave antigua sigue en el historial de git: hay que darla por comprometida y no volver a usarla | No depende técnicamente de H1-C ni de H2-C1. **No** corrige la autorización por correo. **Recomienda** hacerlo antes de H2-D si H2-D mantiene la clave de moderación |
| **H4** *(nivel 1)*. La validación de entrada son `if`s sueltos con códigos y formatos distintos | `routes.ts:310-321` (POST: `400 {error}`, `422 {message}`, `400 {errores}`, `200 {ok:false}`); `routes.ts:383-386` (PATCH sin longitud mínima); `Comentarios.jsx:44` (no lee `errores`) | Quien comenta recibe un mensaje genérico o un 200 cuando el envío falla. Otros clientes de la API reciben códigos distintos según el campo | **3 existentes confirmados:** `routes.ts`, `Comentarios.jsx` y `comentarios.spec.ts`. **1 nuevo:** `app/validators/comentario.ts`. **1 existente condicional:** `exceptions/handler.ts`, solo si el formato por defecto no sirve (sin verificar) | **Sí:** solo es código. El backend y `Comentarios.jsx:44` deben publicarse y revertirse juntos | **Desbloquea** H5: la validación de `padre_id` va en el mismo validador |
| **H1-D** *(nivel 2: depende del nivel 1)*. Transformer de salida para comentarios | Las mismas líneas que H1-C | Ninguna rotura nueva: impide que una columna futura vuelva a filtrarse | 2 y 1 nuevo: `routes.ts`, `comentarios.spec.ts` y el nuevo `app/transformers/comentario_transformer.ts` | **Sí:** solo es código | Requiere tests de caracterización. **Recomienda** hacerlo antes que H6 |
| **H5**. `padre_id` sin validar, sin claves foráneas, y el borrado no trata las respuestas | `routes.ts:308` y `326`; `routes.ts:359`; migración, líneas 6, 14 y 20; `Comentarios.jsx:50` y `63-64` (solo dos niveles) | Una respuesta cuyo padre se borra, o una respuesta a otra respuesta, queda en la base y no se muestra. Lo nota quien la escribió. Hoy: 0 comentarios huérfanos | 3: `routes.ts`, `Comentarios.jsx` y `comentarios.spec.ts`, más el validador de H4. Condicionales: una migración nueva y `blog_seeder.ts` si se añaden claves foráneas | **El código, sí. Una migración ya aplicada, no:** hay que hacer `migration:rollback` con un `down()` correcto. **Las respuestas huérfanas siguen almacenadas. El comentario padre borrado, no:** el borrado es definitivo (`routes.ts:359`) y recuperarlo puede necesitar una copia de seguridad | Depende de H4 (validador) y de una decisión funcional (qué pasa con las respuestas al borrar el padre) |
| **H6**. Tras crear, se lee `order by id desc limit 1` sin filtrar | `routes.ts:329-331` | Si dos POST se cruzan, uno podría recibir el comentario del otro. **No reproducido** | 2: `routes.ts` y `comentarios.spec.ts` | **Sí:** solo es código | Con H1 contenido, la carrera ya no expone correos. **Recomienda** hacerlo después de H1-D |
| **H3** *(nivel 3: bloqueado por una decisión funcional)*. `estado` es texto libre y cada consulta lo interpreta a su manera; el frontend tiene dos contadores | Migración, línea 18; `routes.ts:75-76` y `223` (`<> 'pendiente'`); `routes.ts:153` (`= 'aprobado'`); `blog_seeder.ts:510`, `522` y `575`; `DetallePost.jsx:30`, `38` y `71`; `Comentarios.jsx:11`, `23`, `39` y `54` | El lector ve 4 en el listado y 2 en el detalle. Los comentarios `'Aprobado'` y `'ok'` cuentan pero no se muestran | 5 y 1 nuevo: `routes.ts`, `blog_seeder.ts`, `DetallePost.jsx`, `Comentarios.jsx`, `comentarios.spec.ts` y una migración nueva. `database/schema.ts` se regenera solo | **El código, sí.** **La normalización de datos, solo si la migración guarda el valor original o hay copia de seguridad:** si `'ok'` y `'Aprobado'` pasan a otro valor sin guardarlo, un `down()` no sabe restaurarlos | Bloqueado por la decisión de qué significan `'ok'` y `'Aprobado'`. La normalización va antes que restringir `estado`, y la semilla cambia en el mismo cambio |
| **H2-D**. Modelo de identidad para editar y borrar, con una sola regla de autorización | `routes.ts:348-356` (DELETE: correo sin distinguir mayúsculas, o la clave); `routes.ts:375-381` (PATCH: correo exacto, sin clave, solo si está pendiente) | Cualquiera que conozca un correo puede borrar los comentarios de esa persona. Quien comenta puede borrar pero no editar | Pendiente de la decisión. Opción A, token de edición: `routes.ts`, una migración y un servicio nuevos, y `schema.ts` regenerado. Opción B, moderación con la autenticación que ya existe (`middleware.auth()`): `routes.ts` y un servicio nuevo | **El código, sí.** Con la opción A, la migración necesita `rollback` y los tokens emitidos dejarían de servir | Bloqueado por la decisión de cómo se identifica quien comenta sin cuenta. Sustituye a H2-C1. No depende técnicamente de H1-C |

## 4. Priorización y dependencias

### 4.1 Contención y arreglo definitivo

- **H1 se separa** porque la exposición sigue mientras tanto y no se puede deshacer. H1-C cabe en 2 archivos. H1-D necesita tests de caracterización y una capa nueva.
- **H2 se separa** porque el definitivo depende de una decisión de producto y quizá de una migración. Mientras tanto, un borrado indebido solo se recupera con una copia de seguridad.
- **H2 tiene dos medidas de contención distintas:**
  - **H2-C1** cierra la vía del correo no verificado. Toca 2 archivos y necesita una decisión funcional, porque puede bloquear operaciones legítimas.
  - **H2-C2** cierra la vía de la clave fija. Toca 5 archivos y no tiene coste funcional. No resuelve el problema del correo.
- **Las contenciones de H1 y H2 pueden plantearse en paralelo.** No hay dependencias técnicas entre H1-C, H2-C1 y H2-C2.
- **H3, H4, H5 y H6 no se separan.** No se ha observado un daño continuo e irreversible. En H5, las respuestas sin padre siguen en la base y lo que se pierde es el comentario padre borrado, que es consecuencia del borrado, no de la falta de validación. En H6, además, el arreglo es pequeño.

### 4.2 Dependencias técnicas obligatorias

1. Normalizar `estado` antes de restringirlo (H3): las filas `'Aprobado'` y `'ok'` incumplirían la restricción.
2. Una restricción sobre `estado` obliga a actualizar `blog_seeder.ts` en el mismo cambio. Si no, `db:seed` falla.
3. El nuevo formato de error del backend (H4) y `Comentarios.jsx:44` se publican juntos.
4. Para añadir claves foráneas (H5) no puede haber huérfanos. Hoy hay 0, así que los datos no bloquean.

Entre H1 y H2 no hay dependencias técnicas obligatorias (ver la relación en el apartado 2).

**Por las reglas del proyecto (CLAUDE.md):**
- Tests de caracterización antes de reestructurar.
- Cada cambio de comportamiento es una tarea aparte, con su test.
- Validación en VineJS, nunca en `if`s, por eso H4 va antes que la validación de `padre_id` de H5.

### 4.3 Decisiones funcionales pendientes

- **H2-C:** ¿qué medida de contención es funcionalmente aceptable?
  - H2-C1: restringir el borrado y la edición autorizados solo por correo, a costa de bloquear a quien comenta.
  - H2-C2: cambiar solo la clave, que deja abierta la vía del correo.
  - Las dos.
- **H2-D:** ¿cómo se identifica quien comenta sin cuenta (token, cuenta obligatoria, solo moderación)?
- **H3:** ¿`'ok'` y `'Aprobado'` cuentan como aprobados? ¿Qué valores admite `estado`?
- **H5:** al borrar un comentario con respuestas, ¿se borran en cascada, se rechaza el borrado o se hace un borrado lógico?
- **H4:** ¿qué código y qué formato tiene un error de validación? Pasar del `200 {ok:false}` a otro código cambia el contrato.

### 4.4 Recomendaciones de diseño (no obligatorias técnicamente)

- H1-D antes que H6, para que la respuesta del POST no se toque dos veces.
- H2-C2 antes que H2-D, si H2-D mantiene una clave de moderación.
- Si H3 y H5 reconstruyen la tabla `comentarios`, hacerlo en migraciones separadas y una detrás de otra. Que Lucid reconstruya la tabla en SQLite está sin verificar.
- Hacer en orden todos los cambios de `routes.ts`, que aparece en los seis hallazgos.
- Un solo contador de comentarios en el frontend.
- Pintar hilos de cualquier profundidad. Responder desde la interfaz sería una funcionalidad nueva, no un arreglo.
- Arquitectura completa de CLAUDE.md (controlador, servicio, repositorio, modelo y el alias `#repositories/*` en `package.json`). En este repositorio es obligatoria para los endpoints que se toquen, pero no la exige ninguno de los hallazgos.

### 4.5 Qué se puede deshacer y qué no

| Se puede deshacer | No se puede deshacer |
|---|---|
| Todos los cambios de código, con un revert. En H2-C2, revertir devuelve la clave comprometida | Los correos ya servidos (H1) |
| Una migración, con `migration:rollback`, si su `down()` es correcto (un revert del commit no deshace una migración ya aplicada) | La clave en el historial de git (H2-C2): hay que darla por comprometida, no basta con quitarla del código |
| La normalización de `estado`, si se guarda el valor original o hay copia de seguridad | Los comentarios ya borrados, incluido un comentario padre (H2, H5), salvo que haya copia de seguridad (sin verificar) |
| Las respuestas huérfanas (H5) y los recuentos (H3): el dato sigue en la base | |

### 4.6 Datos que faltan para valorar el daño

- Si hay despliegue en producción y desde cuándo.
- Logs de `GET /posts/:slug`, `DELETE` y `PATCH /comentarios/:id`.
- Si existen copias de seguridad de la base.
- Si el repositorio es o ha sido público.
- Si la clave desplegada es `dev-secret-cambiar`.
- Si hay otros clientes de la API además de este frontend, y si alguien usa hoy el borrado o la edición por correo de forma legítima (dato necesario para decidir H2-C1).
- Cuántos POST concurrentes llegan.
- Las obligaciones legales por exponer correos no se valoran aquí.

## 5. Cierre

1. **Hallazgos propuestos por el agente: 6. Hallazgos inspeccionados manualmente en código: 6.** Eso no significa que se hayan reproducido todas sus consecuencias: H2 se confirmó solo por código y la carrera de H6 no se ha reproducido.
2. **Decisión de prioridad que no podemos resolver todavía:** cómo contener H2. Ni ocultar correos (H1-C) ni cambiar la clave (H2-C2) corrigen la autorización por un correo no verificado, y restringirla (H2-C1) puede bloquear operaciones legítimas. Para decidir faltan datos: si hay despliegue real, logs de `DELETE` y `PATCH /comentarios/:id`, si alguien usa hoy esas operaciones de forma legítima y si existen copias de seguridad. Queda para tu valoración.
3. **Afirmación no verificada:** la posible condición de carrera de H6. El código no liga el select al insert (`routes.ts:329-331`), pero no se ha reproducido que dos peticiones reciban el comentario equivocado.
