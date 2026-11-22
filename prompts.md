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
Audita del proyecto solo la feature de comentarios (crear, editar, borrar y listar), no el sistema completo. 
Limitate a las siguientes capas (no te salgas de ellas):
- rutas API. 
- tablas en la base de datos. 
- lo que expone la web.
La salida será una tabla con una fila por cada hallazgo. Cada fila tendrá 5 columnas con lo siguiente:
- qué es: una frase que indique el hallazgo.
- Dónde está: la línea que lo demuestra. Si no existe línea no lo tengas en cuenta.
- Qué se rompe y quién lo nota: Escribe el efecto y en caso que nadie lo detecte especifica "Nadie todavía" (daño).
- cuantos sitios hay que tocar: No estimes, cuenta los sitios y devuelve la cantidad exacta (radio).
- ¿Bloquea o desbloquea a otro hallazgo?: nombra los hallazgos que bloquea o desbloquea, si no hay pon "A ninguno" 

La salida será un fichero en docs/auditoria/comentarios.md. Si no existe lo creas.
```

**Qué salió:** funcionó a la primera.

## Prompt 2

**Modelo:** Opus 5.5 medium
**Herramienta:** Claude Code

```
Teniendo en cuenta la lista y el orden de ataque que has propuesto vamos a ordenar la lista según la prioridad. Para determinar la prioridad debes tener en cuenta lo siguiente:
- primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes)
- segundo el daño dividido por el radio. Si detectas que un hallazgo tiene el daño más alto y la predencia más baja a la vez indicamelo.
  Actualiza la lista que ya existe en el fichero creado antes. No crees una nueva tabla. Mantén los identificadores de los hallazgos, solo cambia el orden de las filas.
```

**Qué salió:** tuve que insistir.

## Prompt 3

**Modelo:** Opus 5.5 medium
**Herramienta:** Claude Code

```
La forma en la que se determina quién lo nota me parece curiosa. 
Veo sitios donde dice que nadie lo nota pero creo que si se nota, por ejemplo, en el hallazgo C4 dices que nadie lo nota pero un lector crea un comentario con un correo, 
después intenta editarlo con el mismo correo pero que tiene una letra con una mayúscula y no dejará editarlo en cambio si le dejaría borrarlo. Aquí el lector si lo notaría, no? 
Puedes revisar todos estos quién lo nota?
```

**Qué salió:** .

## Prompt 4

**Modelo:** Opus 5.5 medium
**Herramienta:** Claude Code

```
Intenta determinar el valor de la columna Se deshace con un revert. Los posibles valores pueden ser si o no y si es no el motivo.
```

**Qué salió:** funcionó a la primera.
