# Auditoría priorizada: feature de comentarios

> **Cómo usar esta plantilla.** Todo lo que está en bloques `>` como este son instrucciones:
> bórralos cuando entregues. Lo que está fuera de ellos es lo que se rellena.

- **Autor:** Matias Luna (jml)
- **Agente y modelo usados:** Claude Code - Sonnet
- **Alcance:** solo comentarios, en sus tres capas: rutas de la API (`POST /posts/:slug/comentarios`,
  `PATCH /comentarios/:id`, `DELETE /comentarios/:id`, y los comentarios dentro de
  `GET /posts/:slug`), tabla `comentarios` en la base de datos y lo que la web enseña de ellos.
  **Nada más**, aunque el agente se ofrezca a auditar el sistema entero.
- **Tiempo:** puesto el reloj a ___ min (el ejercicio sugiere ~45). **Cuando suene, para.**

---

## Parte A: la tabla de hallazgos

> **Reglas de la tabla**
> - **Una fila por hallazgo.** Si en la casilla 1 necesitas tres frases, son dos hallazgos: sepáralos.
> - **Ordénalos por prioridad**, el más importante arriba (es una auditoría *priorizada*).
> - **Casilla 2 es obligatoria.** Sin `archivo:línea` que lo demuestre, la fila no se escribe.
>   El agente la rellena. Abre el archivo tú y comprueba cuántas líneas dicen lo que se afirma: ese número va en la Parte B.
> - **Si el reloj suena, deja en blanco lo que no te dio tiempo** (escribe `—`). Una fila a medias
>   es información; una fila rellenada de memoria después no lo es. No completes en frío.
> - Contrasta cada hallazgo con las **reglas de negocio** del README (sobre todo la 2, la 5 y la 6):
>   sin regla escrita, un hallazgo es solo una opinión de estilo.

| # | 1. Qué es | 2. Dónde está (archivo:línea que lo demuestra) | 3. Qué se rompe y quién lo nota | 4. Sitios a tocar (contados) | 5. ¿Se deshace con un revert? | 6. ¿Bloquea / desbloquea a otro? |
|---|---|---|---|---|---|---|
| 1 | El detalle del post, el POST y el PATCH devuelven la fila entera del comentario, con el correo de quien comenta (regla 6). | `backend/start/routes.ts:153` (`select * from comentarios ...` → `:171` `comentarios: comentarios`); también `:330`/`:333` (POST) y `:389`/`:392` (PATCH). El frontend no usa `autor_email` en ningún render (`grep` en `frontend/src`: solo se envía en `Comentarios.jsx:33`). | Cualquier visitante lee el correo de todos los que han comentado y quedan expuestos a spam o suplantación. Lo notan los comentaristas, y solo el día que alguien lo explote. | 3 respuestas (GET detalle, POST, PATCH). | Código: sí. Lo ya servido y cacheado o raspado no se recupera. | Lo mitiga #2: sin el correo público, el DELETE de #2 ya no se puede abrir con él. No lo cierra. |
| 2 | Cualquiera puede borrar el comentario de otro: la autorización del DELETE es «traer el correo del autor», y ese correo sale público (#1). | `routes.ts:352-353` (`esAutor = comentario.autor_email ... === email`) + `:350` (el correo llega en el body, `request.input('autor_email')`). | Un visitante sin cuenta borra comentarios ajenos (`:359`) sin dejar rastro. Lo notan los comentaristas, cuando desaparece el suyo. | 1 ruta (el bloque `:349-357`). | Sí el código. El comentario ya borrado no vuelve: no hay borrado lógico ni copia. | Se agrava por #1. Su arreglo de fondo (autenticar al autor) desbloquea el de #6. |
| 3 | La clave de moderación está escrita en el código fuente y es la que da permiso para borrar cualquier comentario. | `routes.ts:25` (`const SECRETO_SESION = 'dev-secret-cambiar'`) y `:349-351` (`clave === SECRETO_SESION`). El `TODO` de `:22-24` admite que debía salir a variables de entorno. | Quien tenga el repo o su historial es moderador y puede borrar lo que quiera. Hoy lo nota el equipo de seguridad, si alguien lo audita. | 2 (la constante y su único uso). Hay que añadir también una variable en `.env`. | No: está en el historial de git. Revertir no la «des-publica», hay que rotarla. | A ninguno. |
| 4 | `estado` es texto libre y las consultas no se ponen de acuerdo: el detalle exige `'aprobado'` exacto y los recuentos cuentan todo lo que no sea `'pendiente'` (regla 5). | Detalle: `routes.ts:153` (`estado = 'aprobado'`). Recuentos: `:75` (listado) y `:223` (relacionados), ambos `estado <> 'pendiente'`. Origen: migración `1769000000005_create_comentarios_table.ts:18` (sin CHECK) y semilla `blog_seeder.ts:13` (conviven `aprobado`, `Aprobado`, `ok`). | En el post 1 el listado dice 4 comentarios (ids 1-4) y el detalle enseña 2 (ids 1 y 2). Los comentarios 3, 4 y 9 son legítimos y no se ven nunca. Lo notan los lectores (números que no cuadran) y los autores de esos comentarios. | 7: 3 consultas, 1 migración (normalizar y poner CHECK) y 3 filas de datos (ids 3, 4, 9). | No: la migración que normaliza los datos pierde el valor original (`ok`, `Aprobado`) y su `down` no puede restaurarlo. | Hay que hacerlo antes de unificar las consultas, o se ocultan más comentarios. Desbloquea #9 (el recuento del front). |
| 5 | Tras insertar, el POST devuelve «la última fila de la tabla», no la que acaba de crear. | `routes.ts:329-331` (`select * from comentarios order by id desc limit 1`). | Si dos personas comentan a la vez, una recibe el comentario de la otra, con su correo (regla 6). Hoy no lo nota nadie (tráfico bajo). Lo notarían los comentaristas con picos de uso. | 1 consulta (`:329-331`). | Sí. | Se agrava por #1. No depende de ninguno. |
| 6 | Las reglas de PATCH y DELETE se escribieron aparte y no coinciden: DELETE compara el correo sin mayúsculas, acepta la clave de moderación y no mira el estado. PATCH compara exacto, no admite al moderador y solo deja editar si está `pendiente`. | DELETE `routes.ts:349-353`; PATCH `:378-379` (`comentario.autor_email !== email \|\| comentario.estado !== 'pendiente'`); el comentario de `:375-377` lo reconoce. | Un moderador puede borrar pero no corregir. Quien escribió «Ana@x.es» puede borrar con «ana@x.es» pero no editar. Un comentario aprobado no se puede editar nunca. Lo notan moderadores y comentaristas. | 2 bloques de comprobación (`:349-353` y `:378-379`). | Sí. | Se arregla junto con #2 (misma regla de «quién es el autor»). |
| 7 | La validación del POST responde con cuatro formas de error distintas, y una de ellas es un `200` que dice que falló. | `routes.ts:310-321`: `:311` `400 {error}`, `:314` `422 {message}`, `:317` `400 {errores:[]}`, `:320` `200 {ok:false,motivo}`. El frontend lo compensa en `Comentarios.jsx:44` (`d.error \|\| d.message \|\| d.motivo`). | Un cliente de la API que mire solo el status cree que un comentario demasiado corto se guardó. Lo notan quienes integren la API. | 4 ramas del backend + 1 línea del frontend = 5. | Sí, pero cambia el contrato de la API: hay que caracterizarlo antes (CLAUDE.md). | A ninguno. |
| 8 | `padre_id` se acepta sin validar y la base no lo protege: puede apuntar a un comentario inexistente, a otro post o a sí mismo, y borrar el padre deja huérfanos a los hijos. | `routes.ts:308` y `:326` (se inserta tal cual); migración `...create_comentarios_table.ts:20` (`padre_id` nullable, sin clave foránea; el aviso está en `:5-6`); `routes.ts:359` (delete sin tocar hijos). | Una respuesta huérfana desaparece de la web: se pinta solo bajo su padre (`Comentarios.jsx:50,63-64`). El front nunca envía `padre_id` (`:33`), así que hoy solo afecta a quien llame a la API a mano o a las respuestas ya existentes (id 2). | 3: ruta POST, ruta DELETE y migración (FK o `ON DELETE`). | Código: sí. Las filas huérfanas que se creen mientras tanto no se limpian solas. | A ninguno. |
| 9 | El frontend lleva dos recuentos de comentarios que no coinciden, y el de la sección suma 1 al enviar un comentario que sigue pendiente. | `Comentarios.jsx:11` (`total`), `:39` (`setTotal(total + 1)`), `:54`; `DetallePost.jsx:30`, `:38`, `:71` (`numComentarios`). Los comentarios `//` de `Comentarios.jsx:8-10` ya lo describen. | El lector envía un comentario y ve «N+1 comentarios» sin que aparezca ninguno nuevo (está pendiente, regla 2). La cabecera sigue en N. Lo nota cualquier lector que comente. | 2 componentes. | Sí. | Le afecta #4 (los números parten ya mal del backend). Conviene resolver #4 antes. |
| 10 | No hay ruta que apruebe un comentario ni interfaz que edite o borre: los comentarios pendientes solo pueden pasar a `aprobado` tocando la base a mano. | `routes.ts:295-393`: solo hay insert, delete y update de `cuerpo` (`:388`). Ninguna sentencia cambia `estado`. `Comentarios.jsx` solo tiene formulario de envío (`:76-88`). | Los comentarios nuevos no se publican nunca por el camino normal. Lo notan los moderadores. **Ojo:** el README no escribe ninguna regla que lo exija, así que puede ser un hueco y no un bug. | 1 ruta nueva + 1 componente (no hay código existente que modificar). | Sí (es solo añadir). | Bloqueado por #3 y #6: no conviene abrir moderación sin resolver quién puede moderar. |
| 11 | La feature casi no tiene tests: hay uno solo, el camino feliz del POST. | `backend/tests/functional/comentarios.spec.ts:15-43` (el comentario de `:5-10` lo confirma). | Nadie todavía. Lo notaría quien cambie cualquier cosa de las filas 1-10 y no se entere de qué rompe. | De 6 rutas que tocan comentarios (POST, PATCH, DELETE, GET detalle, GET listado, GET relacionados) hay 1 con test. | Sí. | Bloquea la corrección segura de #1-#10: CLAUDE.md exige caracterizar antes de cambiar. |
| 12 | Todo el código de comentarios vive en `routes.ts`, con SQL, validación y reglas dentro de la ruta. No hay controlador, validador, servicio, repositorio ni transformer, y la respuesta es la fila cruda. | `routes.ts:295-393` (3 handlers) y `:1-13` (cabecera que lo admite). Contra la regla de arquitectura de `CLAUDE.md`. | Nadie todavía. Es la causa común de #1, #2, #5, #6 y #7. Lo nota quien tenga que cambiar la feature. | 3 handlers, más los archivos nuevos: 1 controlador, 1 validador, 1 servicio, 1 repositorio, 1 transformer y el alias `#repositories/*`. | Sí por pasos, siempre con la caracterización de #11 en verde. | Depende de #11. Resolverlo a la vez que #1 (transformer) y #7 (validador). Es una regla de `CLAUDE.md`, no de negocio. |
| 13 | `comentarios.post_id` no tiene clave foránea ni índice. | Migración `1769000000005_create_comentarios_table.ts:14` (el aviso está en `:5-6`); consultas por `post_id` en `routes.ts:75`, `:153`, `:223`. | Nadie todavía: con 9 filas no se nota. Al crecer, esas tres consultas recorren toda la tabla, y borrar un post dejaría sus comentarios huérfanos. | 1 migración (índice y FK). | Sí el índice. La FK puede fallar si ya hay huérfanos que no he comprobado. | A ninguno. |

> **Cómo se obtuvo esta tabla.** Los 13 hallazgos salen de leer el código (`routes.ts`, migración, semilla, `Comentarios.jsx`, `DetallePost.jsx`, el test) y todas las líneas citadas están leídas en el archivo. **No he ejecutado la API ni la suite**: los efectos de las filas 1, 2, 5 y 9 están razonados desde el código, no reproducidos. Los números «listado 4 / detalle 2» de la fila 4 salen de la semilla (ids 1-4 del post 1).

> **Guía por casilla**
>
> 1. **Qué es.** Una sola frase, sin adjetivos. Ej.: *"El detalle del post devuelve el correo de quien comenta."*
> 2. **Dónde está.** Ruta del archivo + línea + la línea citada. Ej.: `backend/start/routes.ts:123` → `` `SELECT ... email ...` ``.
>    La rellena el agente; tú compruebas las que puedas y anotas cuántas en la Parte B.
> 3. **Qué se rompe y quién lo nota.** Efecto concreto + a quién le llega (lector, autor del post,
>    moderador, nadie). *"Nadie todavía"* es una respuesta válida y buena.
> 4. **Cuántos sitios hay que tocar.** Un número **contado** (con `grep`, no a ojo) y, si ayuda,
>    cuáles: "3: ruta, consulta SQL, componente React".
> 5. **¿Se deshace con un revert?** `Sí` / `No` y, si es no, por qué: típicamente porque hay datos
>    ya escritos en la base de datos, un contrato de API que otros consumen, o una migración aplicada.
> 6. **¿Bloquea o desbloquea a otro hallazgo?** Nombra el otro por su `#`. Ej.: *"Desbloquea #3: hay
>    que validar la entrada antes de poder cambiar la forma del error."* `A ninguno` es lo más común.

---

## Parte B: las tres líneas

> **Esta parte no se puede fallar.** Aunque la tabla quede a medias, estas tres líneas se entregan
> sí o sí. Son cortas y honestas; no las adornes.

1. **Hallazgos devueltos por el agente / comprobados por mí abriendo el archivo:**
   `13 devueltos / 5 comprobados`
   > Los dos números tal cual salieron, aunque sean feos (ej. `14 / 5`). No los redondees ni los iguales.

2. **Hallazgo del que no supe decidir la prioridad, y por qué:**
   `Creo que el orden de prioridad que sugirió el agente es razonable, aunque dejaria los tests para el ultimo, ya que si tocamos algo despues de crear los tests habria que revisarlos y tal vez modificarlos.`
   > Ej.: "#4: no sé si pesa más el riesgo teórico de XSS o que hoy nadie pueda explotarlo."

3. **Algo que el agente afirmó y no pude verificar (ni a favor ni en contra):**
   `No encontre nada que no pueda verificar`
   > No es algo que dijo mal (eso sería un error comprobado). Es lo que quedó en el aire:
   > ej. "Dijo que la tabla no tiene índice en `post_id`, pero no pude ver el plan de consulta."

---

## Antes de entregar

- [ ] Borré todos los bloques `>` de instrucciones.
- [ ] Cada fila tiene `archivo:línea`, y he abierto el archivo en las que puedo afirmar que comprobé.
- [ ] Las casillas que no me dio tiempo a hacer están en blanco o con `—`, no inventadas.
- [ ] Las tres líneas de la Parte B están completas.
- [ ] `prompts.md` relleno (un bloque por prompt, con modelo y herramienta).
