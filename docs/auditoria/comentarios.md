## Parte A

El detalle del post filtra el correo de quien comenta.

Dónde: línea 152, select * from comentarios…, devuelta tal cual en la 171, comentarios: comentarios. Además el POST devuelve la fila cruda en la 333 (comentario: nuevos[0]).
Qué se rompe: cada visitante anónimo recibe autor_email, estado y creado_en de todos los comentaristas, lo que incumple la regla 6. Lo nota cualquiera que abra el JSON; el front no lo muestra.
Sitios a tocar: 3. Son las líneas 152 y 171 (detalle), más la 333 (POST) y un transformer nuevo.
¿Revert? Sí, no cambia datos.
Bloquea o desbloquea: el transformer sirve también al hallazgo 5 y a la respuesta del PATCH (línea 392).
Los comentarios aprobados con otro texto en estado no se muestran.

Dónde: línea 153, estado = 'aprobado'. La semilla tiene 4 'aprobado', 2 'Aprobado' y 1 'ok'. La columna es texto libre (migración …create_comentarios_table.ts, table.string('estado') sin CHECK).
Qué se rompe: 3 comentarios de la semilla desaparecen del detalle, y los lectores y autores no los ven. Es un bug que hoy se oculta solo.
Sitios a tocar: 3 si se corrige. Hay que normalizar los datos con una migración, endurecer la columna y arreglar la consulta de la línea 153.
¿Revert? No. Normalizar 'Aprobado' y 'ok' a 'aprobado' es un UPDATE irreversible, porque el down() no sabe qué filas eran cuáles. Hay que guardar antes la lista de ids afectados.
Bloquea o desbloquea: hay que hacerlo antes que el 3, porque el contador y la lista usan hoy criterios distintos. Hasta resolverlo, cualquier test de caracterización debe fijar el comportamiento actual.
El contador de comentarios cuenta los no aprobados, contra la regla 5.

Dónde: línea 75, c.estado <> 'pendiente' (listado), y línea 223, el mismo criterio en relacionados.
Qué se rompe: el listado y los relacionados dicen N comentarios, y el detalle muestra menos ('Aprobado' y 'ok' cuentan, pero no salen). Además DetallePost.jsx línea 38 y Comentarios.jsx línea 23 recalculan el total con .length. Lo notan los lectores, con números que no cuadran.
Sitios a tocar: 2 en backend (líneas 75 y 223). Subir a 4 si también se quita el recálculo duplicado del front.
¿Revert? Sí. Son solo consultas de lectura.
Bloquea o desbloquea: depende del 2. Si se corrige el contador a = 'aprobado' antes de normalizar los datos, bajan los números mostrados.
El POST falla de forma inconsistente: tres formatos de error y un 200 con fallo.

Dónde: línea 311 devuelve 400 con {error}, la 314 devuelve 422 con {message}, la 317 devuelve 400 con {errores}, y la 320 devuelve 200 {ok:false, motivo}.
Qué se rompe: el cliente tiene que mirar cuatro claves distintas. El front ya lo contorsiona en Comentarios.jsx línea 44, con d.error || d.message || d.motivo. Un cliente que confíe en el código HTTP toma un cuerpo corto como éxito. Lo notan integradores y el propio front.
Sitios a tocar: 4 líneas de la ruta, más un validador nuevo y el handler de excepciones. El front se toca en 1 sitio si cambia el contrato.
¿Revert? Sí. Antes hay que caracterizar el contrato actual (200 {ok:false}, 400/422 con tres formas) con un test.
Bloquea o desbloquea: un validador VineJS desbloquea el 5, porque padre_id se valida en el mismo sitio.
padre_id no se valida y se puede responder a un comentario de otro post.

Dónde: línea 308, request.input('padre_id', null), y línea 326, que lo inserta sin comprobar nada. La migración no tiene índice ni clave foránea sobre post_id ni padre_id.
Qué se rompe: se puede colgar una respuesta de un comentario inexistente o de otro post, y entonces no se muestra nunca (Comentarios.jsx línea 50 solo pinta raíces y sus hijos). Lo nota quien comenta, sin aviso. El front tampoco envía padre_id (línea 32).
Sitios a tocar: 3. Son el validador, la regla en el servicio y la consulta del repositorio. El resto de la ruta queda igual.
¿Revert? Sí para el código. Los huérfanos que ya existan no se arreglan con un revert.
Bloquea o desbloquea: se desbloquea con el 4.
La clave de moderación está escrita en el código y el PATCH aplica otra regla que el DELETE.

Dónde: línea 25, SECRETO_SESION = 'dev-secret-cambiar', usada en la 351 (clave === SECRETO_SESION). El PATCH (línea 379) compara el correo sin normalizar y exige pendiente.
Qué se rompe: cualquiera con el repo puede borrar cualquier comentario con la cabecera x-clave-moderacion. Además, el autor con otras mayúsculas en el correo puede borrar su comentario (línea 353) pero no editarlo. Quien borra con el correo de otro (sin saber más) también puede, porque el correo no es secreto: basta conocerlo, y el POST antes lo expone (hallazgo 1). Lo notan el autor legítimo y la moderación.
Sitios a tocar: 4. Son la constante (a una variable de entorno), el DELETE (línea 351), el PATCH (línea 379) y un servicio de autorización compartido.
¿Revert? No del todo. Si la clave se rota, el revert devuelve la antigua, que ya está comprometida. Y un DELETE ya ejecutado no se deshace: borra la fila físicamente (línea 359).
Bloquea o desbloquea: el hallazgo 1 agrava este (el correo es la credencial de autoría); corregir el 1 lo reduce.

> Funciono con un prompt

## Parte B: las tres líneas



1. 6 el agente, 0 comprobe.
2. No hubo tiempo.
3. No verifique nada.