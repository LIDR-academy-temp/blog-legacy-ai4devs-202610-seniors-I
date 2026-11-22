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

**Modelo:** Opus 5.5 medium
**Herramienta:** Claude Code

```
 Vamos hacer una auditoria de una funcionalidad concreta en esta aplicacion. La funcionalidad a auditar es "Comentarios" Este blog tiene 5
  funcionalidades posts, comentarios, autores, categorías y etiquetas. Centrate unicamente en comentarios  crear,editar, borrar y listar comentarios.
  El formato:
  Por cada hallazgo :
  - Que es: Una frase mas de una indica que es mas de un hallazgo.
  - Donde esta: Linea donde lo encuentras
  - Que rompe y quien lo nota. El efecto de lo que rompe y quien lo nota. Nadie haun es una respuesta valida
  - Cuantos sitions se tiene que tocar para arreglar-lo: No estimes cuenta
  - Se puede deshacer con un revert? Si o  no, en caso de no explica el porque
  - Bloquea o desbloquea otro hallazgo? Nombra al otro. A ninguno es la respuesta mas frequente

  Escribe la auditoria en docs/auditoria/comentarios.md
```

**Qué salió:** Creo el fichero de auditoria.
