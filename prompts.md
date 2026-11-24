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

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
necesito que audites el feature de comentarios en el proyecto actual siguindo las siguientes regglas: 

auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md, no en el chat.

Una feature es una funcionalidad del blog de punta a punta: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella. Este blog tiene cinco (posts, comentarios, autores, categorías y etiquetas), y la tuya es comentarios: crear, editar, borrar y listar comentarios, en esas tres capas. Es la misma que audita el mentor en el directo.

Solo esa feature. No el sistema entero, aunque quepa y aunque el agente se ofrezca.

El formato no es negociable: una fila por hallazgo, con estas seis casillas.

Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.
Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.
Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
Cuántos sitios hay que tocar. Contados, no estimados.
¿Se deshace con un revert? Sí o no, y si es que no, por qué no.
¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

**Qué salió:** funcionó a la primera. Leyó las rutas, la migración, la semilla y los componentes de la web, y comprobó los recuentos contra la BD. Escribió en `docs/auditoria/comentarios.md` una tabla con 30 hallazgos priorizados (P0-P3), un orden de ejecución sugerido y la Parte B con huecos, porque pregunta lo que comprobé yo.

## Prompt 2

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
en comentarios.md luego de la tabla responde lo siguiente Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo. Los dos números, tal cual salieron.
El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué.
Algo que el agente afirmó y no pudiste verificar. No es lo que dijo mal: es lo que no pudiste comprobar ni a favor ni en contra.
```

**Qué salió:** rellenó la Parte B con 30 devueltos / 30 comprobados (solo C07 contrastado contra la BD), C12 como el hallazgo de prioridad dudosa y C03 (la carrera del POST) como lo no verificado. Avisó de que las respuestas son del agente y no una revisión humana.

## Prompt 3

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
agreg en promps.md todos los promps y respuestas que te solicite segun el formato que describe
```

**Qué salió:** borró el ejemplo y añadió estos tres prompts tal cual se lanzaron, con el modelo, la herramienta y una línea de resultado en cada uno.

## Prompt 4

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
si. hacelo. encontre 4
```

**Qué salió:** corrigió la línea 1 de la Parte B con mi número (30 devueltos / 4 comprobados por mí), añadió este prompt, commiteó y abrió el PR.
