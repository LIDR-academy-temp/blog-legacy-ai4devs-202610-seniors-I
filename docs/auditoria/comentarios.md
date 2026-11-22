# Auditoría — feature de comentarios

Alcance: crear, editar, borrar y listar comentarios en API (`backend/start/routes.ts`), tabla
`comentarios` (migración y semilla) y web (`Comentarios.jsx`, `DetallePost.jsx`, `ListaPosts.jsx`,
`PostsRelacionados.jsx`). Contraste: las 6 reglas de negocio del README.

Comprobado leyendo el código, con `git log`/`grep` y con consultas de solo lectura a
`backend/tmp/db.sqlite3`. La API no estaba levantada: no se lanzó ningún GET contra 3401.

| #   | Qué es                                                    | Dónde              | Qué se rompe · quién                               | Sitios | ¿Revert?                     | Bloquea           |
| --- | --------------------------------------------------------- | ------------------ | -------------------------------------------------- | ------ | ---------------------------- | ----------------- |
| 1   | Contención: detalle devuelve `autor_email` de comentarios | routes.ts:153      | Correos ajenos visibles · cualquier visitante      | 1      | Sí                           | Desbloquea #6     |
| 2   | Contención: clave de moderación fija en código            | routes.ts:25       | Con el repo se borra todo · nadie todavía          | 2      | No: clave queda en historial | Desbloquea #6     |
| 3   | `estado` libre; semilla mete `'Aprobado'` y `'ok'`        | migración:18       | Comentarios 3, 4 y 9 ocultos · sus autores         | 4      | No: normalizar cambia datos  | Desbloquea #4, #5 |
| 4   | Recuento cuenta `<> 'pendiente'`; detalle, `= 'aprobado'` | routes.ts:76       | Post 1: portada 4, detalle 2 · lectores            | 3      | Sí                           | Bloqueado por #3  |
| 5   | Web recuenta sola y suma pendientes                       | Comentarios.jsx:39 | +1 por un pendiente · el lector                    | 3      | Sí                           | Bloqueado por #4  |
| 6   | Basta el correo para borrar o editar                      | routes.ts:350      | Cualquiera borra comentarios ajenos · sus autores  | 2      | Sí                           | A ninguno         |
| 7   | Validación a mano, cuatro contratos de error              | routes.ts:310      | Mensaje genérico si falta `@` · quien comenta      | 6      | Sí                           | A ninguno         |
| 8   | POST devuelve el último, no el insertado                  | routes.ts:329      | Recibe comentario ajeno con correo · nadie todavía | 1      | Sí                           | A ninguno         |
| 9   | `padre_id` sin validar ni FK                              | routes.ts:308      | Respuestas huérfanas desaparecen · nadie todavía   | 5      | Sí                           | A ninguno         |

## Notas

1. Devuelve `select *` de comentarios. Rompe la regla 6. Esos correos sirven para borrar (#6): quitarlos abarata #6 porque elimina la credencial pública hoy.
2. Usada en `routes.ts:351` (sitios: `:25` y `:351`). La clave sigue en el historial de git: hay que rotarla. Abarata #6.
3. Sin CHECK. Semilla: `blog_seeder.ts:510`, `:522`, `:575`. Sitios: 4 en código + 3 filas en BD. Git no restaura los datos normalizados.
4. Rompe la regla 5. Líneas `routes.ts:75-76` y `:223` (relacionados), frente a `:153`. Post 6: 1 frente a 0. Sitios: `:76`, `:153`, `:223`. Desbloquea #5.
5. También `Comentarios.jsx:23` y `DetallePost.jsx:38`: la cabecera y la sección dejan de coincidir, y se cuentan las respuestas. Bloqueado por #4 porque el detalle no trae `num_comentarios`.
6. La regla está escrita dos veces y no coincide: DELETE `routes.ts:350-353`, PATCH `:378-379`. Un autor que cambia una mayúscula no puede editar. Sitios: 2 handlers. Contenido por #1 y #2; este es el arreglo de fondo.
7. Líneas `routes.ts:310-321`. El PATCH (`:384`) no exige mínimo y acepta 1 carácter. `Comentarios.jsx:44` muestra "No se ha podido enviar.". Sitios: 5 en la API + 1 en la web.
8. Líneas `routes.ts:329-331`. Pasa con dos envíos a la vez.
9. También `routes.ts:359` (borrar deja hijas), `migración:20` (sin FK), `Comentarios.jsx:50` y `:64` (tampoco pinta el segundo nivel). La web no envía `padre_id`. Sitios: 3 en el backend + 2 en la web. Hoy hay 0 huérfanas.

## Fuera de alcance

- El detalle devuelve el `email` del autor del post (`routes.ts:119`, `:168`, regla 6).
- `/autores/:id` también devuelve ese `email` (`routes.ts:271`).
- El cuerpo del post se inyecta sin sanear (`DetallePost.jsx:92`).
- La URL base de la API está copiada en cada componente (`DetallePost.jsx:7`, `Comentarios.jsx:4`).

## Propuestas sin medir

| #   | Propuesta                                    | Qué medir                                             |
| --- | -------------------------------------------- | ----------------------------------------------------- |
| P1  | Índice en `comentarios.post_id`              | Plan y tiempo de `GET /posts`                         |
| P2  | No repetir `GET /posts/:slug` en Comentarios | Peticiones y latencia por vista de detalle            |
| P3  | Ruta de moderación                           | Pendientes: cuántos, cómo se aprueban, cuánto esperan |
| P4  | Límite de envío o antispam                   | Pendientes por día y parte de spam                    |
| P5  | Consulta agrupada en relacionados            | Tiempo de `GET /posts/:slug/relacionados`             |

### Notas de propuestas

- P1. `migración:14`, sin índice ni FK. Medir con `EXPLAIN QUERY PLAN` la subconsulta de `routes.ts:75`, con volumen real.
- P2. `Comentarios.jsx:19`: hoy salen 2 peticiones iguales.
- P3. Ninguna ruta cambia `estado`.
- P4. El comentario 7 de la semilla es spam.
- P5. `routes.ts:222`: una consulta por relacionado, frente a una sola agrupada.

## Las tres líneas

1. El agente devolvió 9, comprobé 8
   1. El detalle devuelve todas las columnas de los comentarios, autor_email incluido.
   2. La clave de moderación está escrita en el código.
   3. En la base, el estado de los comentarios 3 y 9 es 'Aprobado' y el del 4 es 'ok'. El detalle del post 1 solo devuelve los comentarios 1 y 2.
   4. La portada da 4 comentarios al post 1 y su detalle enseña 2. En el post 6, la portada da 1 y el detalle 0.
   5. frontend/src/componentes/Comentarios.jsx:39: suma 1 al recuento al enviar un comentario pendiente.
   6. Un DELETE del comentario 1 con el correo que enseña el GET público lo borra. Un PATCH con Tomas.iglesias@… en vez de tomas.iglesias@… devuelve 403. El PATCH además solo deja editar comentarios pendientes, y eso no aparece en la fila.
   7. La validación devuelve cuatro errores distintos (400, 422, 400 y 200 con ok:false). El PATCH aceptó el cuerpo "a".
   8. Línea :329-331 confirmada. Pero en 10 pares de envíos simultáneos cada uno recibió su propio comentario. La casilla «Qué se rompe» no la he podido comprobar ni a favor ni en contra.
   9. -
2. Fila 5, «Bloqueado por #4». El motivo que da el agente es que el detalle no devuelve num_comentarios.
3. La fila 8 encaja: el defecto está en el código, pero no hay forma de saber si llega a pasar.
