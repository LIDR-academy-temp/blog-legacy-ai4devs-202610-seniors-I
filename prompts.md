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

## Prompt 1

**Modelo:** Composer (Auto)
**Herramienta:** Cursor Agent

```
Vamos producir la auditoría priorizada de la feature de comentarios y déjarla escrita en docs/auditoria/comentarios.md, no en el chat.

Debes usar este formato:: una fila por hallazgo, con estas seis casillas.





Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.



Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.



Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía"

es una respuesta válida y buena.



Cuántos sitios hay que tocar. Contados, no estimados.



¿Se deshace con un revert? Sí o no, y si es que no, por qué no.



¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta

más frecuente.
```

**Qué salió:** funcionó a la primera; generó 14 hallazgos en la tabla (luego se reordenaron).

---

## Prompt 2

**Modelo:** Composer (Auto)
**Herramienta:** Cursor Agent

```
En la priorida ajustalo con esto, primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño dividido por el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno. Y las propuestas que nadie ha medido todavía van fuera de la tabla, con lo que habría que medir escrito al lado.
```

**Qué salió:** reordenó la tabla (12 en tabla + 4 fuera); partió email, recuento y auth en contención/fondo.

---

## Prompt 3

**Modelo:** Composer (Auto)
**Herramienta:** Cursor Agent (Plan → Agent)

```
Antes de la parte B, vamos a sumar la evidencia, expliquemos porque el punto desbloquea 2,3,7,10 si es un test
```

**Qué salió:** plan + sección «Por qué el 1 desbloquea al 2, 3, 7 y 10» y ajuste de la celda del hallazgo 1.

---

## Prompt 4

**Modelo:** Composer (Auto)
**Herramienta:** Cursor Agent

```
PErfecto ahora hagamos lo mismo par el 2
```

**Qué salió:** sección de evidencia del 2 (abarata al 4; no cierra 10 ni 12) y celda ajustada.

---

## Prompt 5

**Modelo:** Composer (Auto)
**Herramienta:** Cursor Agent

```
HAgamos esta validacion en el hallazgo 3
```

**Qué salió:** sección de evidencia del 3 (corta el daño; no cierra el 11) con el contraste 4 vs 2 del post 1.

---

## Prompt 6

**Modelo:** Composer (Auto)
**Herramienta:** Cursor Agent (Ask mode)

```
Cuales prioridades tienen conflicto y como se resolvieron
```

**Qué salió:** explicación en chat (no tocó el archivo); útil para rellenar la Parte B.

---
