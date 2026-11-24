

## Prompt 1

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Este es el ejemplo. Bórralo.

Audita este repo. Centrate en la feature comentarios, limitate a esta.

Busca 6 afirmaciones, donde se va romper, mostrando la fuente de la misma. Para arreglarlo que se habria que tocar (en que sitios y que lineas).

Mira si al hacer el cambio, y si se modifica informacion se arreglaria con un revert.

Al arreglarlo se bloquearia algo mas?

Dame la salida en el siguient formato de forma priorizada, una linea por hallazgo:
1. **Qué es.** Una frase. .
2. **Dónde está**, con **la línea que lo demuestra**..
3. **Qué se rompe, y quién lo nota.** El efecto concreto y a quién le llega. *"Nadie todavía"* es una respuesta válida y buena.
4. **Cuántos sitios hay que tocar.** Contados, no estimados.
5. **¿Se deshace con un revert?** Sí o no, y si es que no, por qué no.
6. **¿Bloquea o desbloquea a otro hallazgo?** Nombrando al otro. *"A ninguno"* es la respuesta más frecuente.
```

**Qué salió:** Funcionó a la primera sacando lo que se pedía, no lo comprobé, pues me costó mucho saber que pedirle exactamente, aunque la salida tiene su lógica. 
