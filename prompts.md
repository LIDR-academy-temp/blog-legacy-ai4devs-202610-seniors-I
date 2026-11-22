# Prompts

Registro cronológico de los mensajes enviados a Claude Code.

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
Antes de iniciar la auditoría del ejercicio, comprueba que puedes acceder al repositorio.

1. Muéstrame un resumen de los últimos 3 commits mediante Git.
2. Localiza los archivos del proyecto relacionados con la funcionalidad de comentarios.
3. Indica las rutas reales encontradas.

Trabaja exclusivamente en modo lectura. No modifiques archivos, no implementes soluciones y no inicies todavía la auditoría.
````

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
# Prompts - Módulo 6: AI For Legacy

## Prompt 1 - Verificación inicial del repositorio

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```text
Antes de iniciar la auditoría del ejercicio, comprueba que puedes acceder al repositorio.

1. Muéstrame un resumen de los últimos 3 commits mediante Git.
2. Localiza los archivos del proyecto relacionados con la funcionalidad de comentarios.
3. Indica las rutas reales encontradas.

Trabaja exclusivamente en modo lectura. No modifiques archivos, no implementes soluciones y no inicies todavía la auditoría.
````

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
Guárdalo en prompts.md, sustituyendo la plantilla de ejemplo. Conserva literalmente el primer prompt que ejecutamos, con modelo Opus 5.5 y herramienta Claude Code. No hagas commit ni modifiques otros archivos.
````

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
clear
````

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
Vamos a iniciar el ejercicio de auditoría AI For Legacy.

Audita EXCLUSIVAMENTE la funcionalidad de comentarios del Blog Legacy: crear, editar, eliminar y listar, recorriendo frontend, API y persistencia SQLite.

1. Identifica hasta 6 posibles hallazgos concretos.
2. Para cada uno, proporciona archivo real, número de línea y evidencia exacta del código.
3. Clasifica cada afirmación como hecho, riesgo u opinión.
4. Agrupa los problemas transversales que compartan una misma causa.
5. Identifica posibles dependencias entre hallazgos y enumera los archivos afectados cuando puedas comprobarlos.

No inventes evidencias, no asignes severidades arbitrarias y no propongas soluciones sin justificar. Si algo no puede comprobarse, indícalo claramente.

Por ahora presenta los candidatos para mi verificación. No modifiques código, no ejecutes migraciones y no escribas todavía el informe definitivo.
````

## Prompt 6

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
Continuamos la auditoría de comentarios. Ya hemos verificado manualmente H1-H5 y analizado H6, cuya condición de carrera no se ha reproducido.

Antes de priorizar:
1. Para cada hallazgo H1-H6, identifica qué archivos reales habría que modificar para resolver su causa, con ruta y motivo. Cuenta los archivos, no los estimes.
2. Identifica las precedencias reales entre las correcciones: qué arreglo depende de otro y por qué.
3. Distingue las dependencias necesarias de las simples recomendaciones de diseño.
4. Indica qué datos te faltan para valorar el daño y la reversibilidad.
5. Si un hallazgo necesita contención inmediata y arreglo definitivo, propón separarlos y explica por qué.

No asignes puntuaciones ni severidades arbitrarias. No modifiques archivos, no ejecutes cambios y no redactes todavía el informe definitivo. Deja expresamente sin verificar cualquier afirmación que no puedas demostrar.
````

## Prompt 7

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
Vamos a preparar el entregable del ejercicio AI For Legacy.

Crea el archivo `docs/auditoria/comentarios.md` utilizando los seis hallazgos H1-H6 que hemos investigado.

**1. Alcance**

Exclusivamente la funcionalidad de comentarios: frontend, API y persistencia SQLite. No audites otras funcionalidades.

**2. Tabla obligatoria de la academia**

Una fila por trabajo o hallazgo, con exactamente estas seis columnas:

1. Qué es.
2. Dónde está, indicando archivo y líneas reales.
3. Qué se rompe y quién lo nota.
4. Cuántos sitios hay que tocar, enumerándolos.
5. ¿Se deshace con un revert? Justifica la respuesta.
6. ¿Bloquea o desbloquea otro hallazgo? Indica cuál.

Ordena primero por precedencia y, dentro del mismo nivel, por daño en relación con el radio. No utilices puntuaciones ni severidades arbitrarias.

**3. Consideraciones verificadas**

- H1: exposición de `autor_email` confirmada por código y HTTP.
- H2: autorización basada en email y reglas inconsistentes confirmadas por código.
- H3: diferencia entre 4 comentarios contabilizados y 2 devueltos, confirmada por código y HTTP.
- H4: validación fallida que devuelve HTTP 200, confirmada por código y HTTP.
- H5: ausencia de claves foráneas y validación del padre, confirmada. La consulta SQLite encontró 0 comentarios huérfanos.
- H6: consulta que recupera el último comentario global, confirmada por código. La condición de carrera NO ha sido reproducida.

También comprobaste que `SECRETO_SESION` no firma realmente las sesiones. No repitas esa afirmación incorrecta.

**4. Priorización**

Separa H1 y H2 en contención y arreglo definitivo cuando corresponda.

Diferencia dependencias técnicas obligatorias, decisiones funcionales pendientes y recomendaciones de diseño.

No presentes una corrección como irreversible solo por afectar a datos; explica qué puede y qué no puede deshacerse.

**5. Tres líneas obligatorias al final**

Incluye un apartado con:

1. Hallazgos propuestos por el agente: 6. Hallazgos inspeccionados manualmente en código: 6. Aclara que eso no significa que todas las consecuencias estén reproducidas.
2. Decisión de prioridad que no podemos resolver todavía: deja explícita la incertidumbre para que yo la valore personalmente.
3. Afirmación no verificada: posible condición de carrera de H6.

**6. Restricciones**

- No modifiques código fuente, tests ni base de datos.
- No implementes soluciones.
- No hagas commit ni push.
- No modifiques `prompts.md` en este paso.
- Mantén la tabla breve, legible y defendible.
- Si un dato no está demostrado, déjalo pendiente en lugar de inventarlo.

Después de escribir el archivo, muéstrame un resumen de su contenido y ejecuta `git diff -- docs/auditoria/comentarios.md` cuando sea aplicable. Recuerda que los archivos nuevos no aparecen en `git diff` hasta incorporarlos al índice.
````

## Prompt 8

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
Revisa y corrige exclusivamente `docs/auditoria/comentarios.md` según esta auditoría humana del informe.

1. La tabla principal debe tener exactamente SEIS columnas, como exige la academia. Elimina la columna `#` e incorpora los identificadores H1-C, H2-C, etc., en «Qué es».

2. Corrige la relación entre H1 y H2. Ocultar `autor_email` no soluciona la autorización basada únicamente en el correo. No presentes H1-C como una precedencia técnica obligatoria de H2-C o H2-D.

3. Diferencia dos medidas de contención de H2:
   - Restringir temporalmente las operaciones autorizadas solo por un correo no verificado. Señala que requiere decisión funcional porque puede deshabilitar operaciones legítimas.
   - Sustituir la clave de moderación fija por una clave nueva, fuera del código, teniendo en cuenta su presencia en el historial de Git.

   Ambas medidas son distintas y deben tener sus respectivos radios y dependencias. No afirmes que rotar la clave soluciona el problema del correo.

4. En H4, corrige el recuento: 3 archivos existentes confirmados, 1 nuevo y 1 existente condicional.

5. En H5, aclara que las respuestas huérfanas permanecen almacenadas, pero un comentario padre eliminado sí se pierde y podría necesitar una copia de seguridad para recuperarse.

6. Actualiza la priorización y el cierre para reflejar que las contenciones de H1 y H2 pueden plantearse en paralelo. La decisión pendiente de H2 es qué medida de contención resulta funcionalmente aceptable, no si ocultar correos basta para proteger el borrado.

7. Conserva las evidencias ya verificadas, los números de línea, las tres líneas obligatorias del cierre y la incertidumbre de H6. No inventes archivos afectados ni conviertas soluciones opcionales en obligatorias.

No modifiques código, tests, base de datos ni `prompts.md`. No hagas commit ni push.

Al terminar, muestra los apartados modificados y ejecuta `git status -sb`.
````

## Prompt 9

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

````text
Modifica exclusivamente el apartado `## 5. Cierre` de `docs/auditoria/comentarios.md`.

Déjalo en exactamente tres líneas numeradas, sin sublistas:

1. Seis hallazgos propuestos por el agente y seis inspeccionados manualmente en código; aclara que no todas sus consecuencias se reprodujeron.
2. La decisión de prioridad pendiente: cómo contener H2 sin asumir que ocultar correos o rotar la clave resuelve la autorización, y qué información falta para decidir.
3. La afirmación no verificada: la posible condición de carrera de H6.

Conserva el rigor técnico y no cambies ningún otro apartado ni archivo. No hagas commit ni push.
````
