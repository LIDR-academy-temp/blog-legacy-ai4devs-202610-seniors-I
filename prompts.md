# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

---

## Prompt 1 — Preparar el entorno

**Modelo:** MiniMax-M3.1-Flash
**Herramienta:** MCode (MCP local) — Bash, lectura de archivos y navegador integrado (Browser Use)

```
@browser-use
ayudame a preparar el entrno para realiar el ejercicio del modulo 6

2. Forkea y clona el proyecto
Es un blog pequeño con posts, autores, categorías, etiquetas y comentarios, en un monorepo con la API y la web dentro.

Forkea github.com/LIDR-academy-temp/blog-legacy-ai4devs-202610-seniors-I con el botón Fork de arriba, y clona tu fork. Sobre un clon directo del repositorio del curso no tienes permiso de escritura, y aquí vas a crear una rama y commitear.

# 1. Clona TU fork
git clone git@github.com:<tu-usuario>/blog-legacy-ai4devs-202610-seniors-I.git
cd blog-legacy-ai4devs

# 2. Añade el repositorio del curso como "upstream"
git remote add upstream git@github.com:LIDR-academy-temp/blog-legacy-ai4devs-202610-seniors-I.git
git fetch upstream

# 3. Sitúate en la rama de partida, traída del curso
git checkout -b s6/start upstream/s6/start
...
```

**Qué salió:** funcionó con dos desviaciones que hubo que resolver sobre la marcha:

- **El fork no se pudo hacer con el botón de la web.** Se hizo con la API de GitHub
  (`gh api -X POST repos/LIDR-academy-temp/blog-legacy-ai4devs-202610-seniors-I/forks`), que produce
  el mismo resultado: un fork en la cuenta del usuario con `parent` apuntando al repo del curso.
- **SSH no funcionaba** (`Permission denied (publickey)`, sin clave de GitHub registrada en la
  máquina). Se clonó por **HTTPS** con el token de `gh` ya autenticado, y el remoto `upstream` se
  añadió también por HTTPS. El resultado es idéntico; solo cambia la URL que aparece en
  `git remote -v`. La guía avisaba de este fallo ("Si te sale Permission denied (publickey), es SSH
  y no el fork") y el diagnóstico acertó.
- `gh api` falló varias veces con `error connecting to api.github.com` durante la sesión. Resultó
  ser intermitencia de la red, no de la configuración: reintentando funcionó.

## Prompt 2 — Lanzar la auditoría

**Modelo:** MiniMax-M3.1-Flash
**Herramienta:** MCode (MCP local) — Bash, lectura de archivos, navegador integrado (Browser Use)

```
Inicia la auditoría con el agente y registra los hallazgos en docs/auditoria/ con el formato de la guía.

registra los prompts a partir de ahora en prompts.md
```

**Qué salió:** la auditoría se hizo de una sola pasada, leyendo las tres capas y **ejecutando cada
caso** antes de escribirlo. El prompt es corto y no dice ni qué buscar ni cómo; el rigor vino de
leer [`CLAUDE.md`](./CLAUDE.md), el [`README.md`](./README.md) y las 6 reglas de negocio del propio repo, que es lo que fija el
formato de las seis casillas.

Los sub-pasos que el agente se impone a sí mismo en esta fase, por si interesa el rastro:

```
1. Lee backend/start/routes.ts entero, las migraciones y la semilla antes de tocar nada.
2. Lee frontend/src/componentes/comentarios.jsx y DetallePost.jsx.
3. Verifica cada hallazgo candidato ejecutándolo (curl contra la API, sqlite3 contra la base,
   navegador contra la web). Un hallazgo sin evidencia ejecutable no entra en la tabla.
4. Escribe docs/auditoria/comentarios.md con la tabla de seis casillas. Deja la Parte B en blanco:
   las tres líneas las escribe la persona que hace el ejercicio.
5. Escribe prompts.md.
```

---

## Los prompts que fallaron

El README pide incluir los que no funcionaron. Estos dos aparecieron durante el trabajo:

### Fallo 1 — un `grep` que no devolvió nada

**Modelo:** MiniMax-M3.1-Flash
**Herramienta:** Bash

```
grep -rn "comentario" frontend/src --include="*.jsx" --include="*.js" -i
```

```
grep -rn "estado" backend/start/routes.ts | grep -i "update\|set \|aprobar\|rechazar\|moderar"
```

**Qué salió:** el primero funcionó y dio el mapa del frontend. El segundo **no devolvió nada** y
hubo que confirmarlo: la ausencia de resultados de un `grep` no prueba que no exista el código. Se
comprobó de otra forma, listando las rutas que tocan comentarios
(`router.post('/posts/:slug/comentarios')`, `router.delete('/comentarios/:id')`,
`router.patch('/comentarios/:id')`) y confirmando que no hay ninguna que escriba en `estado`. **Ese
"no hay ninguna ruta que modere" es el hallazgo H3, y se sostiene porque se verificó por dos vías.**

### Fallo 2 — un resultado que no cuadraba con lo esperado

**Modelo:** MiniMax-M3.1-Flash
**Herramienta:** Bash (curl)

```
PATCH /comentarios/12   {"autor_email":"Sonda.Trim@Ejemplo.es", ...}
→ 403
PATCH /comentarios/12   {"autor_email":"  sonda.trim@ejemplo.es", ...}
→ 200
```

**Qué salió:** el `403` con mayúsculas y el `200` con espacios parecían contradictorios —si el
PATCH compara el correo "tal cual", los espacios también deberían fallar. No se dio por bueno
ninguno de los dos resultados sin más: se buscó la causa en la configuración
([`backend/config/bodyparser.ts`](./backend/config/bodyparser.ts), que no recorta) y luego se comprobó con un comentario limpio, y se
midió el recorte por separado. **El `trim` lo hace `request.input()`, no el body parser**, y el
recorte de espacios es real: el `PATCH` con espacios al principio o al final responde 200. El
`403` con mayúsculas también es real. La conclusión correcta era más estrecha de lo que parecía:
la diferencia entre PATCH y DELETE es **solo el caso**, no los espacios. Sin esa comprobación, el
hallazgo H6 habría dicho una cosa falsa.

---

## Registro de herramientas

| Fase | Herramienta | Qué se hizo con ella |
|---|---|---|
| Fork y clon | `gh` CLI + `git` | Fork por API, clon por HTTPS, `upstream`, rama `s6/start` desde `upstream/s6/start` |
| Arranque | `make setup` / `make db` / `make api` / `make web` | Dependencias, `APP_KEY`, SQLite sembrada, API en 3401 y web en 5401 |
| Baseline | `make test` | 1 test, en verde. Anotado literal **antes** de tocar nada, como punto de comparación |
| Lectura | `read` + `grep` | [`backend/start/routes.ts`](./backend/start/routes.ts) entero, la migración de comentarios, el seeder, [`Comentarios.jsx`](./frontend/src/componentes/Comentarios.jsx), [`DetallePost.jsx`](./frontend/src/componentes/DetallePost.jsx), el test existente, [`CLAUDE.md`](./CLAUDE.md) |
| Verificación | `curl` + `sqlite3` | 9 endpoints con casos concretos: fuga de correos, 3 criterios de filtrado, PATCH/DELETE, clave de moderación, `padre_id` cruzado, cuerpo corto, índices |
| Verificación visual | Browser Use | Envío real de un comentario por el formulario: la sección pasó a "1 comentarios" con la lista vacía y la cabecera siguió en "0" |
| Escritura | `write` | [`docs/auditoria/comentarios.md`](./docs/auditoria/comentarios.md) y este [`prompts.md`](./prompts.md) |
