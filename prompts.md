# Prompts

Todos los prompts lanzados para hacer el ejercicio, en el orden en que se lanzaron, con el modelo
y la herramienta de cada uno. Reconstruido a partir de `prompt_track.md`.

---

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Vamos a auditar SOLO la feature de comentarios de este blog (crear, editar, borrar y listar comentarios), según el README. No el sistema entero, aunque el agente vea otras cosas.

Reglas para toda la sesión:
- En este paso solo lees: no modificas archivos ni lanzas peticiones que cambien datos (nada de POST, PATCH ni DELETE).
- Cada afirmación lleva archivo:línea. Si no encuentras la línea, lo dices y no escribes el hallazgo.
- Si cuentas algo (sitios, consultas, archivos), lo cuentas de verdad, no lo estimas.
- Distingue siempre entre lo que has visto en el código y lo que deduces.
- El contraste son las 6 reglas de negocio del README. Si un hallazgo no rompe ninguna, dilo.

Paso 1, corte por rutas (capa API):
1. Lista todas las rutas que tocan comentarios: las propias (crear, editar, borrar) y las prestadas (las que devuelven comentarios o su recuento dentro de otra respuesta). Una fila por ruta, con método, ruta, archivo:línea y qué hace con los comentarios.
2. Para cada ruta, indica qué reglas de negocio del README le aplican y si se cumplen, con la línea que lo demuestra.
3. Termina con dos números: cuántas rutas has encontrado y cuántos posibles hallazgos llevas.

Todavía no escribas en docs/auditoria/. Contesta en tablas cortas. Las capas de base de datos y de web irán en los pasos siguientes.
```

**Qué salió:** 6 rutas y 10 posibles hallazgos de la capa API. El agente dijo 4 contra reglas del README, contando dos veces la misma fuga de correos; eran 3 (dos de la regla 5 y uno de la regla 6).

---

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
lee el readme e intentemos hacer la tabla requerida. Yo leí parte del código. Verifiqué que se escapan los mails en la api (llamada mediante curl) y también verifiqué que no se ven en la interfaz web.  Es decir que es un error pero no se ve a simple vista.
```

**Qué salió:** Primera tabla con las 6 casillas del README: 9 hallazgos (C1–C9) tras juntar los dos recuentos.

---

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
revisemos los hallazgos para identificar bien los hechos comprobables.
```

**Qué salió:** Cada hallazgo separado en visto en código / comprobable con GET / solo con escritura / deducción. Recuento respaldado por la semilla y clave localizada en git.

---

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
en particular verifiqué el comentario diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna.  utiliza el orden siguiente de prioridad, verifica que es hallazgo y que es riesgo.  Cuales serían los daños contrastados con las reglas antedichas y las de agents.md. Además mira la precedencia de uno y otro hecho y su alcance/radio de afectación. ordena por precedencia y en segundo lugar por daño  dividido por radio dame una cantidad manejable de hallazgos.
```

**Qué salió:** Reducido a 5 (2 hallazgos, 3 riesgos). No existe AGENTS.md: se contrastó con CLAUDE.md. El agente cuestionó el cociente daño ÷ radio y ordenó por daño.

---

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
para medir el daño necesito que contabilices la afectación y que me digas de las tres capas donde se rompe y puede repararse. Hazme un archivo temporal con las lineas indicando hallazgo, capa  y mostrando un pedazo de código o valor representativo
```

**Qué salió:** Archivo temporal con cada línea por hallazgo y capa. Al leer la web apareció un hallazgo nuevo (H2b).

---

## Prompt 6

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
bien, yo voy a chequear el archivo.  Necesito que contabilices los puntos de afectación, no la cantidad de comentarios. Dame la tabla tal cual la tienes hoy y dime lo que tengo que ver yo según el readme. planteame una estrategia para terminarlo en 10 minutos
```

**Qué salió:** Tuve que corregir la unidad de medida (puntos de código, no comentarios). 27 puntos (24 sin repetir), tabla actual y plan de 10 minutos.

---

## Prompt 7

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Escribe ya docs/auditoria/comentarios.md con la tabla del README (las 6 casillas) para los 6 hallazgos que tienes (H1, H2, H2b, R3, R4, R5), en el orden de prioridad que acordamos.
- Casilla 4: "sitios que hay que tocar para arreglarlo", contados. Pon al lado entre paréntesis los puntos de afectación, para que se vea la diferencia.
- Una casilla que no puedas sostener con una línea se queda vacía. No la rellenes de memoria.
- Debajo de la tabla, deja solo los encabezados de las tres líneas de la parte B, sin rellenarlas: las escribo yo.
No toques ningún otro archivo.
```

**Qué salió:** Archivo escrito con una sola casilla vacía (la 6 de R3).

---

## Prompt 8

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
slug                                               num_comentarios
----                                               ---------------
como-elegir-una-paleta-que-sobreviva-al-rediseno                 0
tipografia-grande-cuando-ayuda-y-cuando-estorba                  0
que-mide-de-verdad-una-metrica-de-producto                       1
los-mejores-portafolios-online-de-diez-disenadores               0
las-dos-caras-del-diseno-de-interfaz-oscuro                      1

  salió en la primera
```

**Qué salió:** El listado está paginado y mi post estaba en la página 2. El agente pidió los GET que faltaban.

---

## Prompt 9

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
para la seguda 

comentarios : {@{id=1; post_id=1; autor_nombre=Ana Cortés; autor_email=ana.cortes@ejemplo.es; cuerpo=La parte de la
              retícula visible me ha convencido. Llevamos dos años escondiéndola y el resultado es que nadie sabe por
              qué las columnas están donde están.; estado=aprobado; creado_en=2026-03-15 10:12:00; padre_id=}, @{id=2;
              post_id=1; autor_nombre=Marta Ruiz Calvo; autor_email=marta.ruiz@corriente.es; cuerpo=Gracias, Ana.
              Justo el mes que viene publicamos un caso con la retícula documentada dentro del propio sitio, a ver qué
              te parece.; estado=aprobado; creado_en=2026-03-15 12:40:00; padre_id=1}}
```

**Qué salió:** H1 confirmado otra vez (2 correos, uno de la autora del post) y mitad de H2 (2 comentarios en el detalle).

---

## Prompt 10

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
slug                                                        num_comentarios
----                                                        ---------------
cinco-consejos-de-diseno-que-aumentaran-tus-conversiones                  0
quince-ejemplos-de-diseno-de-una-sola-pagina                              1
diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna               4
```

**Qué salió:** H2 comprobado en ejecución: 4 en el listado frente a 2 en el detalle.

---

## Prompt 11

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
si
```

**Qué salió:** Prueba de H2 añadida a su casilla 3.

---

## Prompt 12

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
muestrame el faltante de la tabla y reordénalos segun los criterios que mencionamos
```

**Qué salió:** Lista de lo que faltaba y orden estricto por daño ÷ radio, con el aviso de que pone un riesgo por delante de un hallazgo comprobado.

---

## Prompt 13

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
primero la dependencia y luego daño sobre radio.
```

**Qué salió:** Archivo reordenado: R3, H2b, H1, H2 → R4 → R5, con el cálculo encima de la tabla.

---

## Prompt 14

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
recuerda que la casilla h2b la comprobé por web, Contexto de lo que comprobé yo antes de empezar, para que lo tengas en cuenta en la tabla y en la parte B: 1) al enviar un comentario desde la web, queda pendiente pero el contador sube en 1, y al recargar vuelve a su valor; corresponde a H2b (Comentarios.jsx:39), así que márcalo como comprobado por mí en la web, no como deducido. 2) Con curl verifiqué que el detalle del post devuelve el correo de quien comenta y que la web no lo muestra; corresponde a H1 (routes.ts:153), y llegué a él con una pista: el post de `make fuga` del Makefile. 3) Descarté que un comentario aprobado no saliera en la web: estaba mirando otro post, y el error de CORS de la consola solo afecta a la raíz / de la API, no a los comentarios. No cambies nada más del archivo.
```

**Qué salió:** H2b marcado como comprobado en la web. Sin otros cambios.

---

## Prompt 15

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
que hace make fuga?
```

**Qué salió:** Explicación de `Makefile:48-53`, con el matiz de que su `grep` también captura el correo del autor del post, no solo el de los comentaristas.

---

## Prompt 16

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
bien cerremos. que más falta
```

**Qué salió:** Pendientes según el README: Parte B, prompts.md, commit, push y PR, y comprobar la suite antes del commit.

---

## Prompt 17

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
corre la suite para que veas los resultados
```

**Qué salió:** Suite en verde (1 de 1). El primer intento falló por usar el npm de Windows desde WSL.

---

## Prompt 18

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
pude verificar que la api tiene criterio distinto  sobre el uso de valores Aprobado y que no se utiliza enum. Dame candidatos para la parte b
```

**Qué salió:** Candidatos redactados para las tres líneas de la Parte B.

---

## Prompt 19

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Abrí los archivos al ver los extractos por ejemplo en routes.ts las lineas de estados y referencias a comentarios. Finalizó el  análisis  Para 1, 6 devueltos, 4 comprobados
```

**Qué salió:** Línea 1 de la Parte B escrita con mis palabras.

---

## Prompt 20

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
para el 2.  el agregado bug en el comentario parece intrascendente incluso roza la duda si está hecho a drede.
```

**Qué salió:** Línea 2 escrita sobre H2b. El agente añadió una referencia propia (`Comentarios.jsx:8-10`) y lo dijo.

---

## Prompt 21

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
para la 3 R4
```

**Qué salió:** Línea 3 escrita sobre R4. Parte B completa.
