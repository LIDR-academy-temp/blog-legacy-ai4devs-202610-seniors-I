## Parte A: la auditoría, con reloj

*Qué es*: Al crear un comentario, la API no lo vuelve a leer por su id: devuelve el último comentario de la tabla.
*Dónde está*: backend/start/routes.ts:330: 'select * from comentarios order by id desc limit 1'
*Qué se rompe, y quién lo nota*: Si dos personas comentan a la vez, una puede recibir el comentario de la otra, y con él su correo. Nadie todavía: SQLite con poco tráfico, y la web (Comentarios.jsx:38) solo mira si llega d.comentario, no
  lo que contiene.
*Sitios a tocar*: 1 (routes.ts:329-331)
*¿Se deshace con un revert?*: Sí: solo cambia una lectura, no toca datos ni esquema.
*¿Bloquea o desbloquea a otro?*: Lo agrava otro hallazgo: la respuesta devuelve la fila cruda, autor_email incluido (routes.ts:333, regla 6). Si arreglas antes ese, este deja de filtrar correos, aunque sigue devolviendo el comentario
  equivocado.

*Qué es* La insercion de un comentario no comprueba la existencia del padre
*Dónde está* routes.ts::325
*Qué se rompe, y quién lo nota* Nadie lo nota
*Cuántos sitios hay que tocar* En la insercion de un comentario, hay que verificar que el padre existe. Lo mejor a traves de logica y respaldado por un indice en database
*¿Se deshace con un revert?* Si
*¿Bloquea o desbloquea a otro hallazgo?* Bloqueado por "Comentarios huerfanos inconsistencia de db"

*Qué es* Al eliminar un comentario, no es necesario limitar a 1 puesto que id es clave unica
*Dónde está* routes.ts::340
*Qué se rompe, y quién lo nota* Nadie lo nota
*Cuántos sitios hay que tocar* Solo aqui
*¿Se deshace con un revert?* Si, solo es eficiencia SQL
*¿Bloquea o desbloquea a otro hallazgo?* No


*Qué es* Cualquiera puede eliminar el comentario de otro
*Dónde está* routes.ts::353
*Qué se rompe, y quién lo nota* autor_email es una variable que no se verifica y por tanto cualquiera podria introducir el mismo email que el del comentario para borrarlo. Los comentarios de un autor podrian ser eliminados sin su consentimiento
*Cuántos sitios hay que tocar* Varios, porque el email que esta haciendo el delete deberia estar verificado por el servidor en un JWT o en un token de seguridad
*¿Se deshace con un revert?* Si
*¿Bloquea o desbloquea a otro hallazgo?* No


*Qué es* Se permite editar un comentario sin cuerpo
*Dónde está* routes.ts::384
*Qué se rompe, y quién lo nota* Fallo en UI con un comentario donde no hay cuerpo
*Cuántos sitios hay que tocar* Solo aqui
*¿Se deshace con un revert?* Si
*¿Bloquea o desbloquea a otro hallazgo?* No

*Qué es* Comentarios sin post
*Dónde está* 1769000000005_create_comentarios_table::14
*Qué se rompe, y quién lo nota* Un comentario permite pertenecer a un post_id que no existe. Por tanto hay inconsistencia de datos. Afecta al funcionamiento de la app ya que es un error silencioso. Podrian fallar la insercion de comentarios y estos nunca se verian
*Cuántos sitios hay que tocar* En la declaracion de la base de datos
*¿Se deshace con un revert?* No, habria que poner una sentencia de drop index
*¿Bloquea o desbloquea a otro hallazgo?* No

*Qué es* Comentarios huerfanos inconsistencia de db
*Dónde está* 1769000000005_create_comentarios_table::20
*Qué se rompe, y quién lo nota* Un comentario se perderia si su padre_id no existe.
*Cuántos sitios hay que tocar* En la declaracion de la base de datos
*¿Se deshace con un revert?* No, habria que poner una sentencia de drop index
*¿Bloquea o desbloquea a otro hallazgo?* No

*Qué es* Comentarios huerfanos por eliminacion de comentario
*Dónde está* routes.ts::359
*Qué se rompe, y quién lo nota* AL eliminar un comentario, hay que eliminar sus comentarios hijos. Desde dev POV, tendriamos comentarios inaccesibles en bd.
*Cuántos sitios hay que tocar* Donde se ejecute la logica de eliminacion o a traves de la DB usando delete cascade
*¿Se deshace con un revert?* Depende si se quiere hacer por indice o por logica. Yo lo haria por indice y por tanto se necesitaria un drop index
*¿Bloquea o desbloquea a otro hallazgo?* Si, el de los "Los comentarios hijos no se ven"

*Qué es* Los comentarios hijos no se ven
*Dónde está* router.ts::358
*Qué se rompe, y quién lo nota* Los comentarios que descienden de otro comentario no aparecen porque no se consultan. El usuario es el afectado aqui
*Cuántos sitios hay que tocar* En la consulta
*¿Se deshace con un revert?* Si
*¿Bloquea o desbloquea a otro hallazgo?* No

## Parte B: las tres líneas

1. **El agente devolvió 18 hallazgos; lo comprobé abriendo el archivos**
2. **El hallazgo del que no supiste decidir la prioridad»** Veo que las prioridades que ha puesto son correctas 

3. **Algo que el agente afirmó y no pudiste verificar**: 
-  Los errores se improvisan en cada ruta; el gestor central de errores no hace nada propio. │ app/exceptions/handler.ts:15-17. No llego a entender que se hace aqui