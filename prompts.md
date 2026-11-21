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

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 4.8 (claude-opus-4-8)
**Herramienta:** Claude Code

```
Estamos preparando una auditoría académica de Blog Legacy. Sin modificar ningún archivo, resume los últimos 5 commits y localiza los archivos donde se implementa la funcionalidad de comentarios. Por ahora no audites ni propongas soluciones. Confirma que puedes leer el repositorio.
```

**Qué salió:** funcionó a la primera; el historial solo tiene 3 commits (los resumí todos) y localicé los archivos de comentarios en backend y frontend.

---

## Prompt 2

**Modelo:** Opus 4.8 (claude-opus-4-8)
**Herramienta:** Claude Code

```
Antes de iniciar la auditoría, abre el archivo prompts.md y revisa su plantilla.

Registra literalmente este prompt previo de comprobación:

"Estamos preparando una auditoría académica de Blog Legacy. Sin modificar ningún archivo, resume los últimos 5 commits y localiza los archivos donde se implementa la funcionalidad de comentarios. Por ahora no audites ni propongas soluciones. Confirma que puedes leer el repositorio."

Indica la herramienta Claude Code y el modelo que estás utilizando, verificándolo en la sesión si es posible.

Conserva la estructura de la plantilla. No audites todavía, no modifiques código y no hagas commit.

Al terminar, dime cómo quedó preparado el archivo para registrar los siguientes prompts.
```

**Qué salió:** registré el prompt de comprobación como Prompt 1 y dejé la plantilla lista para los siguientes.

---

## Prompt 3

**Modelo:** Opus 4.8 (claude-opus-4-8)
**Herramienta:** Claude Code

```
Vamos a iniciar la auditoría académica del Módulo 6 de AI4DEVS. Tenemos un límite estricto de 45 minutos.

Audita exclusivamente la funcionalidad de comentarios de Blog Legacy, de punta a punta: rutas de API, base de datos y frontend. Incluye crear, editar, borrar y listar comentarios.

Primero identifica posibles hallazgos. No modifiques código ni corrijas defectos.

Para cada candidato, proporciona la evidencia exacta (archivo y línea), explica el comportamiento y distingue hechos comprobables, riesgos derivados y opiniones.

Agrupa los problemas transversales, sin duplicarlos por cada archivo donde aparecen. No inventes evidencia ni asignes severidades arbitrarias.

Trabajaremos con una lista corta y verificaremos personalmente los hallazgos antes de priorizarlos.

Registra este prompt literalmente en prompts.md como Prompt 3, con el modelo y herramienta de esta sesión. Registra también como Prompt 2 el mensaje anterior que te pedía preparar prompts.md, conservando su texto literal si está disponible; si no lo está, indícalo en vez de reconstruirlo.

Por ahora presenta los candidatos y detente. No escribas todavía la auditoría definitiva en docs/auditoria/comentarios.md.
```

**Qué salió:** devolvió 9 candidatos agrupados (C1–C9) con evidencia archivo:línea y se detuvo sin escribir la auditoría, como se pidió.

---

## Prompt 4

**Modelo:** Opus 4.8 (claude-opus-4-8)
**Herramienta:** Claude Code

```
Registra este prompt literalmente en prompts.md, con el modelo y la herramienta de esta sesión.

Crea docs/auditoria/comentarios.md como borrador parcial de la auditoría.

Incluye únicamente los hallazgos C6 y C3 que he comprobado personalmente:

C6: backend/start/routes.ts:153 selecciona todas las columnas de comentarios y :171 las devuelve sin filtrar. Confirmé mediante una petición HTTP al post de ID 1 que dos comentarios aprobados incluyen autor_email.

C3: backend/start/routes.ts:75-76 cuenta estados distintos de pendiente, mientras :153 solo devuelve estado aprobado. Confirmé mediante peticiones HTTP que el post de ID 1 tiene num_comentarios=4, pero el detalle devuelve solamente 2 comentarios.

Para cada hallazgo utiliza las seis columnas obligatorias del ejercicio: qué es, dónde está, daño y afectado, radio contado con sus sitios, reversibilidad y precedencia.

No inventes respuestas. Si una casilla todavía no está comprobada, déjala en blanco.

Deja preparada la sección final de las tres líneas obligatorias, pero no inventes sus respuestas.

No modifiques código, no corrijas los problemas y no hagas commit. Al terminar, muéstrame la tabla.
```

**Qué salió:** creé el borrador parcial en docs/auditoria/comentarios.md con C6 y C3 y las casillas no comprobadas en blanco.

---

## Prompt 5

**Modelo:** Opus 4.8 (claude-opus-4-8)
**Herramienta:** Claude Code

```
Necesito cerrar ahora la entrega del Módulo 6 de AI4DEVS. No tenemos tiempo para más comprobaciones. Usa exclusivamente la evidencia obtenida hasta este momento.

Registra este prompt LITERALMENTE en prompts.md como el siguiente prompt, con modelo y herramienta.

Finaliza docs/auditoria/comentarios.md cumpliendo las instrucciones del README y manteniendo el documento breve, de una sola feature: comentarios.

Evidencia comprobada personalmente:
- C6: backend/start/routes.ts:153 y :171. La API pública del post ID 1 devolvió 2 comentarios aprobados con autor_email visible.
- C3: backend/start/routes.ts:75-76 y :153. La API devolvió num_comentarios=4 en el listado y 2 comentarios en el detalle del mismo post.
- C1: backend/start/routes.ts:349-359. Comprobé leyendo el código que DELETE permite borrar si el correo enviado en el request coincide con autor_email del comentario. No ejecuté ninguna petición DELETE, así que NO afirmes haber demostrado el borrado en ejecución.

El agente generó 9 candidatos (C1-C9). Yo comprobé personalmente 3 hallazgos abriendo código; C6 y C3 también fueron contrastados mediante HTTP. No comprobé los otros 6.

Completa la tabla con las seis casillas oficiales. Para radio, cuenta y enumera solamente sitios que puedas verificar leyendo código. Para reversibilidad, distingue cambios de código de migraciones o modificaciones de datos. Para precedencia, nombra relaciones demostrables; no inventes. Deja en blanco las casillas sin evidencia suficiente.

Ordena por precedencia primero y después por daño respecto al radio, justificando brevemente el orden. Si encuentras un conflicto entre daño y precedencia, distingue contención del arreglo definitivo sin fingir haber implementado ninguno. No agregues problemas no comprobados.

Completa las tres líneas obligatorias de la Parte B:
1. El agente devolvió 9 candidatos; comprobé personalmente 3.
2. Me costó decidir la prioridad entre C1 y C6: C1 permitiría borrar comentarios usando un email aportado por el solicitante, mientras C6 expone ese email públicamente. Quitar el email reduce el riesgo, pero no corrige la falta de autenticación del borrado.
3. No pude verificar en ejecución si una persona sin autenticarse puede borrar un comentario ajeno enviando su correo. Solo revisé la lógica del código.

Si algún dato de estas tres líneas necesita un matiz, consérvalo con honestidad.

No modifiques código de aplicación, pruebas, base de datos ni configuración. Solo docs/auditoria/comentarios.md y prompts.md.

Verifica que esos son los únicos dos archivos modificados, muestra un resumen corto y prepara la entrega. No hagas commit todavía.
```

**Qué salió:** finalicé docs/auditoria/comentarios.md con C1/C6/C3, seis casillas, orden justificado y las tres líneas de la Parte B; solo cambiaron docs/auditoria/comentarios.md y prompts.md.

---

## Prompt 6

**Modelo:** Opus 4.8 (claude-opus-4-8)
**Herramienta:** Claude Code

```
Registra también este prompt literalmente en prompts.md, indicando modelo y herramienta.

Revisa que docs/auditoria/comentarios.md contiene la tabla y las tres líneas obligatorias, y que prompts.md incluye todos los prompts utilizados.

No modifiques ningún otro archivo.

Haz commit únicamente de docs/auditoria/comentarios.md y prompts.md con el mensaje "auditoria: comentarios + prompts".

Publica la rama auditoria-fc en mi fork origin.

Abre UN SOLO Pull Request desde bliqmx:auditoria-fc hacia la rama s6/start del repositorio LIDR-academy-temp/blog-legacy-ai4devs-202610-seniors-I.

Si GitHub no permite abrirlo, no crees otro PR en mi fork como sustituto. Explícame el error y proporciona el enlace para continuar manualmente.

Al terminar, dame la URL del Pull Request y confirma qué dos archivos contiene.
```

**Qué salió:** (pendiente)
