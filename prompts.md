# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

---

## Prompt 1

**Modelo:** Composer
**Herramienta:** Cursor

```
Lee el README y las reglas de negocio. Quiero la auditoría priorizada SOLO de la feature de comentarios (crear, editar, borrar y listar) en API, base de datos y frontend.

Déjala en docs/auditoria/comentarios.md con una fila por hallazgo y estas seis casillas obligatorias:
1. Qué es (una frase)
2. Dónde está, con la línea que lo demuestra
3. Qué se rompe, y quién lo nota
4. Cuántos sitios hay que tocar (contados)
5. ¿Se deshace con un revert? Sí/no y por qué
6. ¿Bloquea o desbloquea a otro hallazgo?

Contrasta contra las reglas del README. No audites posts, autores, categorías ni etiquetas salvo que afecten directamente a comentarios.
Al final añade la Parte B con las tres líneas que pide el ejercicio.
```

**Qué salió:** funcionó a la primera; saqué la tabla y luego comprobé yo los archivos y la API.

---

## Prompt 2

**Modelo:** Composer
**Herramienta:** Cursor

```
Verifica en la API levantada (localhost:3401) si en el detalle de un post salen autor_email de los comentarios, qué status devuelve un comentario con cuerpo corto, y qué trae la respuesta 201 al crear uno válido.
```

**Qué salió:** confirmó la fuga de email, el HTTP 200 con `ok:false` en cuerpo corto, y el 201 con la fila completa incluido `autor_email`.
