# Auditoría — feature de comentarios (Blog «Corriente»)

Feature auditada: **comentarios** (crear, editar, borrar y listar), en sus tres capas: rutas de
la API (`backend/start/routes.ts`), tabla en base de datos (`comentarios`) y lo que la web
enseña de ellos. Solo esta feature.

Alcance de la evidencia: el agente generó 9 candidatos (C1–C9); aquí se incluyen únicamente los
**3 comprobados personalmente**. C6 y C3 se contrastaron además por HTTP contra el post de ID 1;
C1 se revisó leyendo el código, **sin ejecutar ninguna petición DELETE**. Las casillas sin
evidencia suficiente se dejan en blanco a propósito.

## Parte A: la auditoría

Orden: por precedencia primero, luego por daño respecto al radio (ver justificación bajo la tabla).

| # | 1. Qué es | 2. Dónde está (línea que lo demuestra) | 3. Qué se rompe, y quién lo nota | 4. Cuántos sitios hay que tocar | 5. ¿Se deshace con un revert? | 6. ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C1 | `DELETE /comentarios/:id` borra un comentario si el `autor_email` enviado en el request coincide con el del comentario; no hay autenticación de por medio. | `backend/start/routes.ts:349-359` (identidad de "autor" en `:350-353`; borrado en `:359`). | **Según la lógica del código (no ejecutado):** cualquiera que aporte el email del comentarista podría borrar su comentario sin autenticarse. Lo notarían los comentaristas (comentarios ajenos borrables). *No verificado en ejecución.* | 1 sitio (1 archivo): el bloque de autorización del handler DELETE, `routes.ts:349-357` + el `delete` en `:359`. *(El mismo patrón de autorización por email reaparece en `PATCH` `:378-381`, pero es otro hallazgo, no incluido aquí.)* | Sí. El arreglo sería lógica de autorización (**cambio de código**); no implica migración ni modificación de datos, así que un revert lo deshace. | Relación demostrable con **C6**: C1 usa como identidad el `autor_email` que **C6** expone públicamente. C6 agrava a C1; no lo bloquea técnicamente (C1 sería explotable si el email se conoce por otra vía). |
| C6 | La respuesta pública del detalle del post expone el `autor_email` de los comentarios. | `backend/start/routes.ts:153` (`select *` de comentarios) devuelto sin filtrar en `:171`. | El correo de cada comentarista sale en el JSON público. **Confirmado por HTTP:** el post de ID 1 devolvió 2 comentarios aprobados con `autor_email` visible. Lo nota cualquier consumidor de la API (dato personal expuesto). | 3 sitios (mismo archivo), verificables leyendo código: detalle `routes.ts:152-153`→`:171` (confirmado por HTTP); `POST` `:329-333` (devuelve la fila nueva con `select *`); `PATCH` `:389-392` (devuelve la fila con `select *`). | Sí. Dejar de devolver el campo es **cambio de código**; no toca migración ni datos. *(Matiz: el dato ya divulgado no se "des-divulga"; eso es ajeno al revert.)* | Agrava a **C1** (expone el email que C1 usa como prueba de identidad). Quitar el email es **contención** de C1, no su arreglo. |
| C3 | Dos criterios distintos de "aprobado": el recuento cuenta un conjunto y el listado del detalle devuelve otro. | `backend/start/routes.ts:75-76` cuenta `estado <> 'pendiente'`; `:153` solo devuelve `estado = 'aprobado'`. | El recuento público y los comentarios mostrados no coinciden. **Confirmado por HTTP:** el post de ID 1 reporta `num_comentarios = 4` pero el detalle devuelve 2 comentarios. Lo nota el lector que compara la cifra con la lista. | 2 sitios confirmados: `routes.ts:75-76` y `:153`. Leyendo código, el criterio del recuento (`<> 'pendiente'`) se repite en `:223` (recuento en `/relacionados`): 3.º sitio. | Parcial. Unificar el criterio en código es **cambio de código**, reversible con revert. Normalizar los valores de `estado` en la columna sería **migración/datos**, que un revert **no** deshace. | A ninguno demostrable entre los hallazgos de este documento. |

**Justificación del orden.** C1 y C6 van primero por su relación de precedencia **demostrable**
(C6 expone el `autor_email` que C1 usa como identidad). Entre ellos, **C1 encabeza por daño**
(borrado de comentarios ajenos sin autenticación) aunque su arreglo definitivo —autenticación—
sea de mayor radio; **C6 le sigue como contención** barata (dejar de exponer el email, 3 sitios
de código) que **reduce** el riesgo de C1 pero **no lo cierra**. Aquí está el conflicto entre
daño y precedencia: el máximo daño (C1) no se resuelve con el cambio más barato (C6); C6 contiene,
C1 corrige. **C3 va al final**: sin relación de precedencia y con daño acotado a la coherencia del
recuento. *(No se ha implementado ninguna corrección ni contención; esto describe el plan, no un
cambio hecho.)*

## Parte B: las tres líneas

1. **Hallazgos devueltos por el agente vs. comprobados personalmente:** el agente devolvió
   **9 candidatos** (C1–C9); comprobé personalmente **3** (C1, C3, C6). C3 y C6 se contrastaron
   además por HTTP; C1 solo por lectura de código.
2. **Hallazgo cuya prioridad costó decidir:** entre **C1 y C6**. C1 permitiría borrar comentarios
   usando un email aportado por el solicitante, mientras C6 expone ese email públicamente. Quitar
   el email (C6) reduce el riesgo, pero **no** corrige la falta de autenticación del borrado (C1).
3. **Algo afirmado y no verificado:** no pude verificar **en ejecución** si una persona sin
   autenticarse puede borrar un comentario ajeno enviando su correo. Solo revisé la lógica del
   código; no se ejecutó ninguna petición DELETE.
