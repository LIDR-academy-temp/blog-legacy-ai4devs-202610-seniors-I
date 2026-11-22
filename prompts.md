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

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code

```
Audita la feature de COMENTARIOS de este blog y deja la auditoría priorizada escrita en `docs/auditoria/comentarios.md`. No la imprimas en el chat: al terminar, en el chat solo di la ruta del archivo y cuántas filas tiene la tabla.

Usa UN solo agente para hacer la auditoría. No lances más agentes ni repartas el trabajo en paralelo.

## Alcance (estricto)
Feature = comentarios: crear, editar, borrar y listar, en sus tres capas:
1. Rutas de la API (handlers, validación, autorización, servicios de comentarios).
2. Tabla(s) de comentarios en la base de datos (esquema, índices, constraints, migraciones, queries).
3. Lo que la web muestra de comentarios (componentes, servicios HTTP, formularios, estados de error y carga).

Solo esa feature. No audites posts, autores, categorías ni etiquetas, ni el sistema entero, aunque quepa en contexto. Si un hallazgo de comentarios depende de código compartido (middleware de auth, un helper común), puedes citarlo como evidencia, pero el hallazgo debe ser sobre comentarios. Si ves algo fuera de alcance, ignóralo; no lo anotes.

## Reglas de trabajo
- Solo lectura. Lo único que escribes es `docs/auditoria/comentarios.md` (crea comentarios.md). No modifiques código, esquema ni configuración.
- Hallazgo = afirmación concreta que otra persona podría ir a comprobar. Si no se puede comprobar, es una impresión y no entra.
- **REGLA DURA: un hallazgo sin la línea que lo demuestra no es un hallazgo.** Cada fila lleva `ruta/archivo.ext:línea` (o rango) donde se ve el defecto. Antes de escribir una fila, abre ese archivo, lee esa línea y confirma que el código dice lo que la fila afirma. Si no lo puedes demostrar con una línea, la fila NO se escribe.
- Cuida los falsos positivos: un hallazgo que suena bien y no lo es cuesta lo mismo de verificar que uno real. Para cada candidato, intenta refutarlo (¿hay una validación, un constraint o un middleware en otro punto que lo cubre?). Si lo refutas, descártalo.
- Si un defecto aparece repetido porque nace de una sola decisión (transversal), es UN hallazgo y sus repeticiones son el radio, no filas separadas.
- Una frase por "Qué es". Si necesitas tres, son dos hallazgos: sepáralos.

## Formato (no negociable)
Una tabla Markdown, una fila por hallazgo, con exactamente estas seis columnas, en este orden:

| # | 1. Qué es | 2. Dónde está (archivo:línea) | 3. Qué se rompe y quién lo nota | 4. Cuántos sitios hay que tocar | 5. ¿Se deshace con un revert? | 6. ¿Bloquea o desbloquea a otro hallazgo? |

- (1) Una frase.
- (2) Ruta y línea que lo demuestran.
- (3) Efecto concreto y a quién le llega (usuario final, autor del comentario, admin, operación, nadie). "Nadie todavía" es una respuesta válida.
- (4) El radio se CUENTA, no se estima. Escribe el número y la lista de sitios (`archivo:línea`). Búscalos con grep/lectura, no de memoria. "Varios sitios" no vale. Si no puedes listarlos, no está medido: va fuera de la tabla.
- (5) "Sí" o "No". Si es "No", di por qué (migración de datos, dato ya corrompido, cambio de contrato con clientes, etc.).
- (6) Nombra el hallazgo concreto (por su #). "A ninguno" es válido y es la respuesta más frecuente; no inventes dependencias para llenar la casilla.

## Orden de la tabla
1. **Primero la precedencia.** Lo que abarata o desbloquea a otro hallazgo va antes que lo que no. Agrupa en niveles: Nivel 1 = desbloquea/abarata a otros; Nivel 2 = independiente; Nivel 3 = depende de otros o se abarata si otro se hace antes. Deriva el nivel de la casilla 6; deben ser coherentes.
2. **Dentro de cada nivel, daño ÷ radio**, de mayor a menor. Daño: escala 1–5 (1 = cosmético; 3 = degrada la experiencia o el dato de forma recuperable; 5 = pérdida/corrupción de datos, fallo de seguridad o caída). Radio = el número contado de la casilla 4. Muestra daño, radio y cociente al final de cada fila o en una columna extra pequeña, para que el orden sea auditable.
3. **Regla de partición:** si un hallazgo tiene el daño más alto y a la vez la precedencia más baja (Nivel 3), NO lo dejes como una fila. Pártelo en dos: (a) una **contención** que corta el daño hoy (mínima, con su propio radio contado y su propia reversibilidad) y (b) el **arreglo de fondo**, que espera su turno según su precedencia real. Marca ambas filas como par y enlázalas en la casilla 6.

## Fuera de la tabla
Las propuestas o sospechas que nadie ha medido todavía (no se pudo contar el radio, o no hay línea que las demuestre aún) NO van en la tabla. Ponlas en una sección aparte, "Propuestas sin medir", cada una con lo que habría que medir escrito al lado (qué contar, qué query o prueba ejecutar, qué archivo abrir).

## Estructura del archivo
1. Título y fecha.
2. Alcance en 3 líneas (qué capas se leyeron: lista de archivos revisados).
3. Tabla priorizada (con niveles como subencabezados).
4. Propuestas sin medir.
5. Descartados: candidatos que refutaste, con una línea de por qué (para no repetirlos).

## Comprobación final (antes de terminar)
Relee tu propio archivo y verifica: (a) toda fila tiene `archivo:línea` y la línea fue abierta; (b) toda casilla 4 tiene número y lista; (c) ningún hallazgo es de otra feature; (d) el orden respeta precedencia y luego cociente; (e) los pares contención/fondo están enlazados. Corrige lo que falle. Luego responde en el chat solo con la ruta y el número de filas.
```

**Qué salió:** He escrito la auditoría en docs/auditoria/comentarios.md. La tabla priorizada tiene 19 filas: 4 en el Nivel 1, 8 en el 2 y 7 en el 3. Las filas #5 y #16 son un par contención/fondo.

---

## Prompt 2

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code

```
Estoy de acuerdo con los hallazgos, pero la tabla priorizada debe ser mejorada dada la instrucción "la tabla priorizada debe caber en una pantalla"; genera la propuesta en un nuevo doc "comentarios_v2"
```

**Qué salió:** He creado la propuesta en docs/auditoria/comentarios_v2.md. La tabla priorizada ocupa 25 líneas: cabecera, 3 filas de nivel y 20 hallazgos. Las filas más anchas tienen 162 caracteres en el texto en bruto; renderizada, cabe en una pantalla normal. Este es el documento aprobado que subo como comentarios.md en el ejercicio.
