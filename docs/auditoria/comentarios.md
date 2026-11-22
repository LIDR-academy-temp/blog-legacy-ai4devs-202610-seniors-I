# Auditoría — feature de comentarios

**Alcance:** crear (`POST /posts/:slug/comentarios`), editar (`PATCH /comentarios/:id`),
borrar (`DELETE /comentarios/:id`) y listar (dentro de `GET /posts/:slug` y los recuentos
`num_comentarios`). Solo tres capas: rutas API, tabla `comentarios` y lo que expone la web.

**Convenciones**

- Rutas abreviadas: `routes.ts` = `backend/start/routes.ts`;
  `mig_comentarios.ts` = `backend/database/migrations/1769000000005_create_comentarios_table.ts`;
  los `.jsx` viven en `frontend/src/componentes/`.
- **Radio**: número de sitios que hay que tocar en el código actual. Un sitio es una línea o un
  bloque de líneas contiguas, y todos aparecen entre paréntesis para poder recontarlos.
- **Bloquea X**: X no se puede cerrar del todo mientras este hallazgo siga abierto.
  **Desbloquea X**: arreglar este hallazgo resuelve X o lo deja en un solo sitio.
- **Quién lo nota**: la primera persona que percibe el efecto cuando el escenario ocurre, por
  cualquier canal que exista hoy, sea la web o una llamada directa a la API. Hay que tener en
  cuenta que **la web solo crea comentarios**: el único `fetch` con método es el POST de
  `Comentarios.jsx:31`. Editar y borrar solo se puede llamando a la API. «**Nadie todavía**» se
  reserva para efectos que ocurren pero que hoy ninguna persona puede percibir.
- **¿Se deshace con un revert?**: si el arreglo de este hallazgo sale mal, ¿basta un `git revert`
  para dejar el sistema exactamente como estaba, sin efectos que queden? «Sí» cuando el arreglo
  solo toca código. «No» cuando deja datos o esquema que el revert no restaura: un `git revert`
  no ejecuta `node ace migration:rollback`, y un `down()` no puede recuperar valores que el
  arreglo sobrescribió o borró.
- **Las filas están ordenadas por prioridad.** El criterio se explica al final del documento.

| # | Qué es | Dónde está | Qué se rompe y quién lo nota (daño) | Radio | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C18 | Crear, editar y borrar viven enteros en `routes.ts`, con SQL crudo, `if`s de validación y sin transformer. | `routes.ts:295`, `routes.ts:339`, `routes.ts:366` | Ningún usuario ve un efecto directo, pero cada regla (autorización, validación, forma de la respuesta) se ha copiado a mano y ha salido distinta (C3/C4, C12/C14). **Lo nota el equipo de desarrollo** cuando cambia una regla y tiene que hacerlo en tres sitios. | 3 (`routes.ts:295`, `routes.ts:339`, `routes.ts:366`) | Sí | Desbloquea C1, C2, C4, C12 |
| C12 | La validación del POST da cuatro errores con tres códigos (400, 422 y 200) y cuatro formas de cuerpo (`error`, `message`, `errores`, `motivo`). | `routes.ts:311`, `routes.ts:314`, `routes.ts:317`, `routes.ts:320`; la web los recoge en `Comentarios.jsx:44` | Con un correo sin `@`, la respuesta es `{errores:[…]}`, que `Comentarios.jsx:44` no lee, así que la web muestra «No se ha podido enviar.». **Lo nota el lector que comenta**, que no sabe qué corregir. | 5 (`routes.ts:311`, `routes.ts:314`, `routes.ts:317`, `routes.ts:320`, `Comentarios.jsx:44`) | Sí | Desbloquea C13, C14, C15 |
| C1 | El detalle del post devuelve cada comentario aprobado con `select *`, incluido `autor_email`. | `routes.ts:153` (`select * from comentarios …`) y `routes.ts:171` (`comentarios: comentarios`) | Los correos de quienes comentan salen en una respuesta pública, lo que incumple la regla 6. Con esos correos, C3 permite borrar cualquier comentario aprobado. **Lo nota quien mire las respuestas de la API**, por ejemplo con las herramientas de red del navegador, y puede explotarlo. Quien comentó no lo ve, porque la web no pinta el correo. | 2 (`routes.ts:153`, `routes.ts:171`) | Sí | Bloquea C3 |
| C3 | Para borrar basta con mandar en el cuerpo el mismo correo que el comentario. | `routes.ts:350-353` (`request.input('autor_email')` comparado con `comentario.autor_email`) | Quien conozca el correo de alguien, o lo saque por C1, puede borrar sus comentarios. **Lo nota quien comentó**, al ver que su comentario ha desaparecido de la web. | 1 (`routes.ts:350-353`) | Sí | Bloquea C4 |
| C5 | Después de insertar, el POST devuelve el último comentario de la tabla, no el que acaba de crear. | `routes.ts:330` (`select * from comentarios order by id desc limit 1`) | Si llegan dos POST a la vez, uno recibe el comentario del otro, con su correo (C2). **Lo nota quien llama a la API directamente** y lee la respuesta. Desde la web no se ve, porque `Comentarios.jsx:37-38` solo comprueba que exista `d.comentario` y muestra «Gracias». | 1 (`routes.ts:329-331`) | Sí | Bloquea C2 |
| C8 | `estado` es texto libre, sin CHECK ni enum, y ya contiene 'Aprobado' y 'ok'. | `mig_comentarios.ts:18` (y el aviso en `mig_comentarios.ts:4-6`) | Los comentarios aprobados como 'Aprobado' o 'ok' nunca salen en el detalle (`routes.ts:153`) pero sí cuentan en el listado (C7). **Lo nota quien comentó**, porque su comentario aprobado no aparece. | 1 (`mig_comentarios.ts:18`, con una migración nueva que normalice los valores y restrinja la columna) | No: el arreglo necesita una migración que reescribe datos, y `git revert` quita el archivo pero no ejecuta `migration:rollback`. La base se queda migrada y `adonis_schema` registra una migración que ya no existe. Además, normalizar a 'aprobado' es destructivo: ningún `down()` puede saber qué filas eran 'Aprobado' y cuáles 'ok' (los tres valores conviven, `blog_seeder.ts:13`). | Bloquea C7 |
| C6 | La clave de moderación está escrita en el código y es la misma que firma las sesiones. | `routes.ts:25` (`SECRETO_SESION = 'dev-secret-cambiar'`) y `routes.ts:351` | Cualquiera que lea el repo puede borrar cualquier comentario con la cabecera `x-clave-moderacion`. **Lo nota quien comentó**, al ver que su comentario ha desaparecido de la web. | 2 (`routes.ts:25`, `routes.ts:351`) | Sí | Bloquea C4 |
| C10 | `padre_id` no tiene clave foránea y la ruta lo acepta sin validar. | `mig_comentarios.ts:20` y `routes.ts:308` (`request.input('padre_id', null)`) | Por la API se puede responder a un comentario inexistente o de otro post; la web no envía `padre_id` (`Comentarios.jsx:33`). Aunque se apruebe, esa respuesta no sale nunca en la web, porque no es raíz (`Comentarios.jsx:50`) y ninguna raíz la reclama (`Comentarios.jsx:64`). **Lo nota quien escribió la respuesta**, porque nunca aparece. | 2 (`mig_comentarios.ts:20`, `routes.ts:308`) | No: el arreglo necesita una migración que reescribe datos, y `git revert` quita el archivo pero no ejecuta `migration:rollback`. La base se queda migrada y `adonis_schema` registra una migración que ya no existe. Además, para añadir la clave foránea hay que recrear la tabla y anular o borrar los `padre_id` inválidos, y esos valores no vuelven. | Bloquea C11 |
| C17 | La web lleva dos recuentos de comentarios y pide el post dos veces. | `DetallePost.jsx:38`, `Comentarios.jsx:19`, `Comentarios.jsx:23` | Cada artículo hace dos `GET /posts/:slug` y guarda dos contadores que se separan en cuanto alguien comenta (C16). **Lo nota el lector que acaba de comentar**, porque ve dos números distintos en la misma página. La petición duplicada no la nota nadie. | 3 (`DetallePost.jsx:38`, `Comentarios.jsx:19`, `Comentarios.jsx:23`) | Sí | Desbloquea C16 |
| C11 | Al borrar un comentario padre, sus respuestas se quedan en la tabla. | `routes.ts:359` (`delete from comentarios where id = ?`) | Las respuestas aprobadas siguen en la base, pero desaparecen de la web (`Comentarios.jsx:50`, `Comentarios.jsx:64`) y siguen contando en el listado (C7). **Lo nota quien escribió la respuesta**, que ve desaparecer un comentario que nadie ha borrado. | 1 (`routes.ts:359`) | No: el arreglo natural borra las respuestas en cascada, y las que se borren mientras esté activo no vuelven con el revert. Solo sería Sí con un arreglo que no borre, como impedir el borrado o reasignar las respuestas. | A ninguno |
| C16 | Al enviar, la web suma 1 al recuento aunque el comentario quede pendiente y no se muestre. | `Comentarios.jsx:39` (`setTotal(total + 1)`) | El título dice «N+1 comentarios» pero la lista muestra N, y la cabecera del artículo (`DetallePost.jsx:71`) sigue en N. **Lo nota el lector que acaba de comentar.** | 1 (`Comentarios.jsx:39`) | Sí | A ninguno |
| C2 | Crear y editar devuelven la fila cruda: `autor_email`, `estado`, `post_id` y `padre_id`. | `routes.ts:333` (`comentario: nuevos[0]`) y `routes.ts:392` (`comentario: actualizados[0]`) | Expone el correo y campos internos que el contrato no pide; con C5, el correo puede ser de otra persona. **Lo nota quien llama a la API directamente.** Desde la web no se ve, porque descarta el contenido de la respuesta (`Comentarios.jsx:37`). | 2 (`routes.ts:333`, `routes.ts:392`) | Sí | A ninguno |
| C4 | Editar usa otra regla de autorización: compara el correo distinguiendo mayúsculas, no acepta la clave de moderación y solo deja editar comentarios pendientes. | `routes.ts:379` | Quien comentó con `Ana@x.com` puede borrar su comentario mandando `ana@x.com`, pero no editarlo. Tampoco puede editarlo una vez aprobado, y un moderador puede borrar pero no editar. **Lo nota quien comentó o quien modera**, al editar por la API, porque recibe un 403 donde el borrado sí funciona. La web no tiene opción de editar. | 2 (`routes.ts:350-353`, `routes.ts:379`) | Sí | A ninguno |
| C7 | Los recuentos cuentan todo lo que no está `pendiente` (`<> 'pendiente'`), pero el detalle solo muestra `= 'aprobado'`. | `routes.ts:75-76` (listado), `routes.ts:223` (relacionados) frente a `routes.ts:153` (detalle) | La portada (`ListaPosts.jsx:57`) y los relacionados (`PostsRelacionados.jsx:32`) dan más comentarios que la cabecera del artículo (`DetallePost.jsx:71`), porque cuentan los de `estado` 'Aprobado', 'ok' o cualquier otro valor. Incumple la regla 5. **Lo nota el lector.** | 2 (`routes.ts:75-76`, `routes.ts:223`) | Sí | A ninguno |
| C9 | `post_id` no tiene clave foránea ni índice. | `mig_comentarios.ts:14` | Si se borra un post directamente en la base, sus comentarios quedan huérfanos; la API no tiene ninguna ruta para borrar posts. La subconsulta de recuento del listado (`routes.ts:75`) recorre la tabla entera por cada post. **Nadie todavía**: con los datos de hoy no hay lentitud apreciable. Con volumen, lo notaría el lector en una portada lenta. | 1 (`mig_comentarios.ts:14`) | No: el arreglo necesita una migración que reescribe datos, y `git revert` quita el archivo pero no ejecuta `migration:rollback`. La base se queda migrada y `adonis_schema` registra una migración que ya no existe. Además, SQLite no puede añadir una clave foránea con `ALTER TABLE`: hay que recrear la tabla y borrar o reasignar los comentarios huérfanos, y esas filas no vuelven. El índice, por sí solo, sí sería reversible. | A ninguno |
| C13 | Un cuerpo de menos de 5 caracteres devuelve `200 {ok:false}`. | `routes.ts:320` | La web se salva porque comprueba `d.comentario` (`Comentarios.jsx:37`). Un cliente de la API que mire el código de estado da el comentario por creado, y como un comentario nuevo queda pendiente, que no aparezca no le extraña. **Nadie todavía**: el error es silencioso por diseño. | 1 (`routes.ts:320`) | Sí | A ninguno |
| C14 | Editar no aplica el mínimo de 5 caracteres que sí exige crear. | `routes.ts:384` (`if (!cuerpo)`) frente a `routes.ts:319` | Por la API se puede dejar un comentario en «a». **Nadie todavía**: solo se pueden editar comentarios pendientes (`routes.ts:379`) y no hay ninguna pantalla de moderación, así que ninguna persona lo ve hasta que se apruebe. | 1 (`routes.ts:384`) | Sí | A ninguno |
| C15 | El correo solo se valida comprobando que contenga una `@`. | `routes.ts:316` (`indexOf('@')`) | Acepta «@» o «a@» como correo válido. **Nadie todavía**: la web no vuelve a usar el correo, y solo cuenta como credencial en editar o borrar por la API, donde quien comentó tendría que repetir el mismo valor inválido. | 1 (`routes.ts:316`) | Sí | A ninguno |

## Parte B: las tres líneas

1. Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo: 18 hallazgos devueltos - 10 hallazgos verificados.
2. El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué: Algunos hallazgos que tienen baja precedencia y un daño similar C13, C14 y C15 se han ordenado según el identificador porque no fui capaz de detectar algo que pudiera dar un orden entre estos 3 hallazgos. El orden se ha hecho por el identificador.
3. Algo que el agente afirmó y no pudiste verificar: No he podido verificar todos los que por tiempo no he podido. Por ejemplo C17, C11, C16,... 


## Criterio de prioridad

1. **Precedencia.** Para cada hallazgo se cuentan los hallazgos que desbloquea, directa o
   indirectamente, siguiendo la última columna. Cuantos más desbloquea, antes va: C18 = 8
   (C1, C2, C3, C4, C12, C13, C14, C15), C12 = 3, C1 = 2 (C3, C4), C3 / C5 / C6 / C8 / C10 / C17
   = 1, y el resto = 0.
2. **Daño ÷ radio**, para desempatar dentro del mismo nivel de precedencia. El daño se puntúa así:
   - **3**: fuga de datos personales o borrado sin autorización real (C1, C3, C6).
   - **2**: algo que una persona usuaria ve mal o pierde, o una fuga que puede llegar a
     terceros (C2, C4, C5, C7, C8, C10, C11, C12, C16, C17).
   - **1**: lo nota solo el equipo, o nadie todavía (C9, C13, C14, C15, C18).

   Si el cociente empata, va antes el de mayor daño absoluto y, después, el de identificador menor.

El orden resultante también respeta todas las dependencias: ningún hallazgo aparece antes de otro
que lo bloquee.

Cociente daño ÷ radio de cada hallazgo: C3 3,00 · C5 2,00 · C8 2,00 · C11 2,00 · C16 2,00 ·
C1 1,50 · C6 1,50 · C2 1,00 · C4 1,00 · C7 1,00 · C10 1,00 · C9 1,00 · C13 1,00 · C14 1,00 ·
C15 1,00 · C17 0,67 · C12 0,40 · C18 0,33.

**Tensiones del orden**

- Ningún hallazgo combina a la vez el daño más alto (3) y la precedencia más baja (0).
- C3 tiene el mejor cociente de daño ÷ radio (3,00), pero queda cuarto. Va detrás de C18 y C12,
  que tienen más precedencia, y detrás de C1, que lo bloquea.
- C11 y C16 tienen cociente 2,00, pero su precedencia es 0, así que caen a los puestos 10 y 11.
- C18 es el caso contrario: tiene el peor cociente (0,33) y va primero solo por precedencia.
