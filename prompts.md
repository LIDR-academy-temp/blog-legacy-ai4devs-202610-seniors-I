# Prompts

Todos los prompts que lancé para hacer el ejercicio, en orden, con modelo y herramienta.
El ejercicio manda a Claude; en mi caso la herramienta es **OpenCode**.

---

## Prompt 1

**Modelo:** DeepSeek V4.1 Flash
**Herramienta:** OpenCode

```
necesito resolver todo el ejercicio ubicado aqui: https://github.com/LIDR-academy-temp/blog-legacy-ai4devs-202610-seniors-I, adicional mi repo principal de git es https://github.com/ElShifuDeveloper, mi nombre es Pedro Molina Suazo, porfavor asume que lo que dice que se ahga con claude se hace con Opencode
```

**Qué salió:** leyó el README y dedujo el ejercicio, el fork y la rama; el repo del enunciado
solo existe como `LIDR-academy-temp/…` (el `LIDR-academy/…` del README da 404), así que el fork
y el PR van ahí.

---

## Prompt 2

**Modelo:** DeepSeek V4.1 Flash (subagente «adversarial-reviewer»)
**Herramienta:** OpenCode

```
Trabajas en el repo /Users/pedro.molina/Documents/DESARROLLO/IA/blog-legacy-ai4devs-202610-seniors-I (rama auditoria-pms). Es un blog legacy: AdonisJS 7 + SQLite en backend/, React 19 + Vite en frontend/. Toda la lógica del blog vive en backend/start/routes.ts.

TAREA: auditoría priorizada de la feature de COMENTARIOS y SOLO de comentarios (crear, editar, borrar, listar), en sus tres capas: rutas de la API, tabla de base de datos y lo que la web enseña de ella. Nada más del sistema.

Reglas de negocio que el sistema DEBERÍA cumplir (están en el README y en CLAUDE.md):
1. Un post en borrador no aparece en listados, relacionados, recientes ni búsqueda, ni para su autor.
2. Un comentario pendiente no se muestra en el detalle del post.
3. slug es único entre posts.
4. Los relacionados nunca incluyen el propio post y salen como mucho N.
5. El recuento de comentarios cuenta SOLO los aprobados.
6. La respuesta pública de un post NO expone el correo del autor ni el de quien comenta.

Lee, sin saltarte ninguno: backend/start/routes.ts entero, backend/database/migrations/1769000000005_create_comentarios_table.ts, backend/database/seeders/blog_seeder.ts (los COMENTARIOS y los estados raros), frontend/src/componentes/Comentarios.jsx, frontend/src/componentes/DetallePost.jsx, frontend/src/componentes/ListaPosts.jsx, backend/tests/functional/comentarios.spec.ts. Contrasta con backend/app/controllers/, backend/app/validators/user.ts y backend/app/transformers/user_transformer.ts, que son el único ejemplo del repo escrito en la arquitectura correcta.

Devuelve UNA FILA POR HALLAZGO, priorizada, con estas seis casillas exactamente:
1. Qué es (una frase; si necesitas tres, son dos hallazgos).
2. Dónde está, con la LÍNEA que lo demuestra (archivo:línea). Sin esta casilla la fila no vale.
3. Qué se rompe y quién lo nota (efecto concreto y a quién le llega; "nadie todavía" es válido).
4. Cuántos sitios hay que tocar, contados, no estimados.
5. ¿Se deshace con un revert? Sí o no, y si es que no, por qué.
6. ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro, o "a ninguno".

Exige rigor: no inventes vulnerabilidades que no puedas señalar con una línea, no cuentes estilo como hallazgo, no propongas arreglos ni escribas código. Distingue bug de contrato. Si algo es un hueco de tests, es un hallazgo más.

Formato de salida: tabla markdown + una lista final "descartado y por qué" con lo que consideraste y no llegó a fila. Responde en castellano.
```

**Qué salió:** 16 filas con prioridad y tipo, más 15 descartes; me corrigió una cifra mía (el
post 1 tiene 4 comentarios no pendientes, no 3) y admitió que el `select` final no lo pudo
reproducir.

---

## Prompt 3

**Modelo:** DeepSeek V4.1 Flash (subagente «adversarial-reviewer», misma sesión)
**Herramienta:** OpenCode

```
He recibido tu tabla. Segunda pasada, solo de verificación. No repitas la auditoría entera.

1. Re-sembré la base (make db) y medí: el post 1 anuncia 4 comentarios en el listado y el detalle enseña 2; el post 6 anuncia 1 y enseña 0. Confirma o corrige con la evidencia que sacaste tú.

2. Evalúa estas cuatro filas que no están en tu tabla, con el mismo criterio (línea que lo demuestra, efecto concreto, sitios contados, revert):
   - La cabecera del artículo (frontend/src/componentes/DetallePost.jsx:38,71) y la tarjeta del listado (frontend/src/componentes/ListaPosts.jsx:57) dan números distintos del mismo post.
   - El `200 {ok:false}` del cuerpo corto, como hallazgo propio y no solo como formato de error.
   - El `PATCH` valida menos que el `POST` (backend/start/routes.ts:383-386 frente a `:319`).
   - El detalle devuelve todos los aprobados sin límite ni paginación (backend/start/routes.ts:152-155).

3. Sobre tu fila del `select` final (`order by id desc limit 1`): di si mantienes el hallazgo sin haberlo reproducido o lo retiras, y con qué evidencia.

4. Lista, en tres líneas, lo que afirmaste y NO pudiste verificar.

No toques archivos del repo. Responde solo a esos cuatro puntos, en castellano.
```

**Qué salió:** confirmó las cifras y rechazó como fila propia la cabecera contra la tarjeta (la
explica entera el recuento); aceptó el `200 {ok:false}` como hallazgo aparte y por su propia
regla partió en dos la fila de errores; mantuvo el `select` final pero degradado.

---

## Lo que no lleva prompt

- **Fork, rama, `make setup` / `make db` / `make test`, commit y PR**: los ejecuté yo desde la
  terminal. Ningún agente hizo esos pasos, así que no hay prompt que enseñar.
- **La Parte B**: la escribí yo con lo que salió de la verificación, sin pedírsela a nadie.
- **El alcance del Prompt 2 tiene un fallo mío**: le pasé las seis reglas del README y tres son
  de posts. Por ahí entró la fila 13 (`slug` sin `unique`). La dejé porque decide dónde aterriza
  un comentario, pero es el único hallazgo cuyo arreglo toca otra feature, y como desliz de
  alcance es mío, no del agente.