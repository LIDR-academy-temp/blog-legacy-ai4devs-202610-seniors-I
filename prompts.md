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

## Prompts

**Modelo:** Sonnet
**Herramienta:** Claude Code

## Prompt 1

```
lee el @README.md y explicame el ejercicio, tanto la parte A como la parte
  B. Create tambien el archivo docs/auditoria/comentarios.md con una
  plantilla con explicacion de como lo tengo que completar
```

## Prompt 2

```
 si a la tabla de hallazgos la tiene que completar el agente, por que me
  decis que la columna arhivo:linea tiene que ser comprobada por mi? Hay
  datos de la tabla que tengo que completar a mano yo?
```

**Qué salió:** Me aclaro que la parte A la hace el agente

## Prompt 3

```
 Necesito que hagas la parte A del ejercicio, en la que rellenas la tabla de hallazgos del archivo @docs/auditoria/comentarios.md. Tanto en el @README.md como en el archivo
  @docs/auditoria/comentarios.md hay instrucciones sobre como llenarla. Recuerda que vamos a auditar solo la feature de comentarios, incluyendo:
  - API: crear, editar y borrar comentarios, y los comentarios dentro del detalle del post.
  - Base de datos: la tabla comentarios.
  - Web: lo que el frontend enseña de ellos.
```

## Prompt 4

```
 me generas un DER, un diagrama de base de datos asi veo de forma simple las tablas, su estructura y su relacion? (Puede ser en mermaid o cualquier grafico simple)
```

## Prompt 5

```
 En la parte B me dice que tengo que decir que hallazgo no supe priorizar, pero que tarea me dice que tengo que priorizar la lista?
```
