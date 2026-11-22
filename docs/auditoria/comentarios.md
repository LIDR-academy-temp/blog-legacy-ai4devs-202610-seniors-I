# Auditoría de comentarios

**Fecha:** 2026-11-22.

**Alcance.** API: `backend/start/routes.ts` (POST, DELETE, PATCH, detalle y recuentos). BD: migración `1769000000005_create_comentarios_table.ts`, `blog_seeder.ts` y `tmp/db.sqlite3` abierta en solo lectura. Web: `Comentarios.jsx` y `DetallePost.jsx`.

**Leyenda.** R = `backend/start/routes.ts` · C = `frontend/src/componentes/Comentarios.jsx` · M = migración de comentarios · S = `backend/database/seeders/blog_seeder.ts`. La columna «Sitios» da solo el número; la lista contada está en el anexo. D/R es daño ÷ radio. «Dep.» indica qué hallazgo bloquea o desbloquea a cuál.

## Tabla priorizada

| # | Qué es | Dónde | Qué se rompe → quién | Sitios | ¿Revert? | Dep. | D/R |
|---|---|---|---|---|---|---|---|
| | **Nivel 1: desbloquea o abarata** | | | | | | |
| 1 | DELETE no re-parenta las respuestas | R:359 | Las respuestas aprobadas desaparecen → lector y autor | 1 | Sí | Desbloquea 16 | 3/1=3,00 |
| 2 | El detalle devuelve `autor_email` (`select *`) | R:153 | Hay 4 correos públicos, 2 en el post 1 → quien comenta | 4 | Sí | Desbloquea 18 | 5/4=1,25 |
| 3 | `estado` está sucio: `Aprobado` ×2 y `ok` ×1 | S:510 | 3 de los 7 aprobados están ocultos → lector | 4 | No (datos) | Desbloquea 15, abarata 17 | 4/4=1,00 |
| 4 | Validación a mano y débil en POST y PATCH | R:319 | Aceptan `"     "`, `"@"` y `"a"` → moderador | 3 | No (contrato) | Abarata 14 y 19 | 2/3=0,67 |
| 5 | La web pide el post dos veces | C:19 | 14 consultas por vista en vez de 7 → operación | 2 | Sí | Abarata 20 | 1/2=0,50 |
| | **Nivel 2: independientes** | | | | | | |
| 6 | **Par 6/18, contención:** DELETE solo con la clave | R:352 | Cierra el borrado con el correo filtrado en 2 → autor | 1 | Sí | Par de 18 | 5/1=5,00 |
| 7 | El POST devuelve la última fila y no la suya | R:330 | Con concurrencia, comentario ajeno (sin reproducir) → quien comenta | 1 | Sí | — | 4/1=4,00 |
| 8 | La web no lee `errores` | C:44 | Un email sin `@` da un error genérico → quien comenta | 1 | Sí | — | 3/1=3,00 |
| 9 | El contador suma 1 al enviar un comentario pendiente | C:39 | Muestra «N+1 comentarios» con N visibles → lector | 1 | Sí | — | 2/1=2,00 |
| 10 | La clave de moderación está en el código | R:25 | Quien lea el repo puede borrar cualquiera → autor | 4 | Sí | — | 5/4=1,25 |
| 11 | El formulario no tiene estado «enviando» | C:86 | Un doble clic crea 2 pendientes → moderador | 2 | Sí | — | 2/2=1,00 |
| 12 | `post_id` no tiene índice | M:14 | `SCAN` en cada recuento → nadie todavía | 1 | Sí | — | 1/1=1,00 |
| 13 | Fecha en UTC pintada sin convertir | C:59 | Sale el día anterior entre las 00 y las 01–02 h → lector | 2 | Sí | — | 1/2=0,50 |
| | **Nivel 3: dependen de otros** | | | | | | |
| 14 | `padre_id` no se valida | R:308 | La respuesta se guarda y no se pinta → autor | 1 | Sí | Depende de 4 | 3/1=3,00 |
| 15 | `estado` no tiene CHECK | M:18 | Puede volver a ensuciarse → nadie todavía | 1 | No (tabla) | Depende de 3, desbloquea 16 | 2/1=2,00 |
| 16 | No hay FK en `post_id` ni `padre_id` | M:20 | Puede haber huérfanos (hoy 0) → nadie todavía | 1 | No (tabla) | Depende de 1 y 15 | 2/1=2,00 |
| 17 | Los recuentos usan `<> 'pendiente'` | R:223 | El listado dice 4 y el detalle muestra 2 → lector | 2 | Sí | Depende de 3 | 3/2=1,50 |
| 18 | **Par 6/18, fondo:** se autoriza por correo | R:353 | Borra o edita quien sepa el correo → autor | 5 | No (contrato) | Par de 6, depende de 2 | 5/5=1,00 |
| 19 | 4 formas de error, una con estado 200 | R:320 | Un fallo responde 200 → clientes de la API | 5 | No (contrato) | Depende de 4 | 3/5=0,60 |
| 20 | La web no tiene estado de carga ni de error | C:25 | Muestra «0 comentarios» si tarda o falla → lector | 4 | Sí | Depende de 5 | 2/4=0,50 |

---

## Anexo A · Sitios contados

| # | v1 | Sitios |
|---|---|---|
| 1 | #7 | R:359 |
| 2 | #1 | R:152-155 · R:329-333 · R:389-392 · `app/transformers/comentario_transformer.ts` (nuevo). Debe conservar la envoltura `comentario` que lee C:37. |
| 3 | #2a | migración nueva (UPDATE que normaliza) · S:510 · S:522 · S:575 |
| 4 | #3 | R:305-321 · R:383-386 · `app/validators/comentario.ts` (nuevo) |
| 5 | #4 | C:17-26 · `DetallePost.jsx:123` |
| 6 | #5 | R:352-355 |
| 7 | #6 | R:329-331 |
| 8 | #18b | C:44 (se solapa con 19) |
| 9 | #8 | C:39 |
| 10 | #9 | R:25 · `backend/start/env.ts:22` · `backend/.env.example:9` · `backend/.env` (no versionado; `Makefile:27` no lo regenera) |
| 11 | #11 | C:28-34 · C:86 |
| 12 | #14b | migración nueva (índice en `post_id`) |
| 13 | #12 | C:59 · C:68 |
| 14 | #13 | R:308 (regla `exists` en el validador de 4) |
| 15 | #2b | migración nueva (CHECK, con reconstrucción de tabla) |
| 16 | #14a | migración nueva (FK con `ON DELETE` coherente con 1; misma reconstrucción que 15) |
| 17 | #15 | R:75-76 · R:223 |
| 18 | #16 + #17 | migración nueva (token) · R:323-327 · R:333 · R:350-355 · R:378-381. Absorbe la divergencia DELETE/PATCH de v1 #17 (R:351-353 frente a R:379). |
| 19 | #18a | R:311 · R:314 · R:317 · R:320 · C:44 |
| 20 | #19 | C:11 · C:20 · C:25 · C:54 |

## Anexo B · Cambios respecto a v1

- **Cifras corregidas.** #2: son 4 correos en total y 2 en el post 1, no «4 en el post 1». #6: la fila afecta a los `aprobado` exactos. #10: el radio pasa de 3 a 4 (`backend/.env`). #13: la franja es 00–01 h en invierno y 00–02 h en verano.
- **Filas partidas.** v1 #2 → 3 (datos) + 15 (CHECK, que no se puede añadir con datos sucios). v1 #14 → 12 (índice) + 16 (FK). v1 #18 → 8 (web, Nivel 2) + 19 (API).
- **Fusión.** v1 #17 entra en el radio de 18, porque es la misma decisión.
- **Dependencias.** v1 #7 sube al Nivel 1 como fila 1. better-sqlite3 aplica las FK, así que una FK en `padre_id` sin `ON DELETE` rompe R:359 y la fila 16 depende de la 1.
- **Sale de la tabla.** v1 #10 (rate limit) pasa al anexo C: su radio no se puede contar sin instalar el paquete.
- **No se parte.** v1 #3 se queda entera porque tiene un solo validador como arreglo. v1 #13 también: si la API exige que el padre sea raíz, el límite de un nivel en la web deja de alcanzarse.
- **Sigue sin reproducir.** La carrera de la fila 7. La línea la demuestra, pero el efecto con concurrencia no se ha probado.

## Anexo C · Propuestas sin medir

| Propuesta | Qué medir |
|---|---|
| El POST no tiene rate limit (R:295, sin `@adonisjs/limiter`) | Ejecutar `node ace add @adonisjs/limiter` en una rama y contar los archivos que toca. |
| La carrera de la fila 7 | Test con 2 POST en `Promise.all` que compruebe que cada respuesta trae su `autor_nombre`. |
| El detalle no pagina los comentarios (R:152-155) | En producción, `select post_id, count(*) from comentarios group by 1 order by 2 desc limit 10`. |
| Hay clientes externos que hacen «No» los reverts de 4, 18 y 19 | Logs de las rutas de comentarios agrupados por `Origin` y `User-Agent`, excluyendo `localhost:5401`. |
| Hay más variantes de `estado` o huérfanos en producción | `select estado, count(*) … group by estado` y los `left join` de huérfanos (`comentarios-verificacion.md`, Q1 y Q8). |
| Tamaño máximo de `cuerpo` | El valor por defecto de `limit` JSON en `@adonisjs/bodyparser`, o un POST de 2 MB. |
| Zona horaria de las fechas de la semilla (afecta a 13) | Preguntar al origen del dato o comparar con logs de creación. |

## Anexo D · Descartados

Siguen vigentes los de `comentarios.md`: XSS en el cuerpo, inyección SQL, pendientes visibles, comentar en un borrador, CSRF, PATCH solo sobre pendientes, comparación de la clave sin tiempo constante, ids no numéricos, correo vacío y huérfanos actuales.

## Parte B

1. **Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo:** 20 y 12.
1. **El hallazgo del que no supiste decidir la prioridad:** 7, la probabilidad de que dos POST coincidan en el mismo segundo/milisegundo es baja, otros hallazgos son más grave/urgente.
1. **Algo que el agente afirmó y no pudiste verificar:** 1, no disponía de comentarios con respuestas para simular hallazgo, si se materializa puede generar molestia en "los comentaristas" de las respuestas.