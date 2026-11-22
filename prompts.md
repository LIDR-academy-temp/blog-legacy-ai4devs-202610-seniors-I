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

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Estoy levantando este proyecto con `make setup`, `make db`, `make api` y `make test`. Me sale este error: `<pega el error>`. Mi entorno es `<macOS / Linux / WSL>` con Node `<versión>`. Dime la causa más probable y el comando exacto para arreglarlo. No modifiques archivos del repo.
```

**Qué salió:** Lo envié por error, con la plantilla sin rellenar y antes de haber ejecutado los make. El agente avisó de que no había ningún error pegado, revisó la máquina sin modificar nada (Fedora 43, Node v24.22.1) y encontró que backend/.env y frontend/.env apuntaban al puerto 3333 en vez de al 3401. Corregí los dos .env a mano con sed y relancé make api y make web


## Prompt 2 

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Lee README.md, CLAUDE.md y .claude/commands/refactor.md. Resúmeme en 5 viñetas como mucho: (1) las reglas de negocio numeradas, (2) la arquitectura objetivo, (3) qué hace el hook guard-tests. No propongas cambios ni empieces a auditar.
```

**Qué salió:** 
```
- Reglas de negocio (1–3): (1) Un post en borrador no aparece en listados,
    relacionados, recientes ni búsqueda, tampoco para su autor. (2) Un comentario
    pendiente no se muestra en el detalle del post. (3) El slug no se repite entre
    posts.
  - Reglas de negocio (4–6): (4) Los relacionados nunca incluyen el propio post y
    salen como mucho N. (5) El recuento de comentarios cuenta solo los aprobados. (6)
    La respuesta pública de un post no muestra el correo del autor ni el de quien
    comenta.
  - Arquitectura objetivo: la petición pasa por estas capas: ruta fina (solo verbo,
    path y [Controlador, 'metodo']) → controlador fino → validador VineJS (toda la
    validación de entrada) → servicio (toda la regla de negocio y el único sitio que
    cambia datos) → repositorio con Lucid (todo el acceso a datos, alias
    #repositories/* por crear) → modelo, que nunca se devuelve al cliente. La
    respuesta siempre sale por un transformer. Los errores se gestionan en un solo
    sitio, app/exceptions/handler.ts.
  - Hook guard-tests (qué hace): se ejecuta antes de cada comando Bash del agente. Si
    el comando contiene git commit, corre npm test en backend/ y, si la suite falla,
    bloquea el commit (sale con código 2) y devuelve al agente las últimas 30 líneas
    de la salida. Como vigila la herramienta Bash y no el hook de git, --no-verify
    tampoco lo salta.
  - Hook guard-tests (umbral de cobertura): la constante COBERTURA_MINIMA está en 0 a
    propósito, así que hoy solo exige que la suite esté verde y ni siquiera mide la
    cobertura. Si se sube por encima de 0, corre npm run test:coverage y bloquea el
    commit cuando la cobertura de línea queda por debajo. Se propone subirlo a 40,
    luego a 60 y al final a 80.
```

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Haz un inventario de TODOS los archivos que pertenecen a la feature de comentarios  
  del blog, en sus tres capas. Solo esa feature: ignora lo que sea exclusivo de       
  posts, autores, categorías o etiquetas.                                             
                                                                                      
  Busca en:                                                                           
  - API (backend/): las rutas POST /posts/:slug/comentarios, PATCH /comentarios/:id y 
  DELETE /comentarios/:id en el archivo de rutas, sus controladores, y la parte de    
  GET /posts/:slug que carga, filtra, serializa o cuenta comentarios.                 
  - Datos (backend/): el modelo de comentario, sus migraciones, seeders o factories,  
  y cualquier validador, serializer, transformer o servicio que toque comentarios.    
  - Web (frontend/): los componentes, páginas, hooks o llamadas a la API que crean,   
  editan, borran, listan o cuentan comentarios.                                       
  - Tests: los que cubren comentarios.                                                
                                                                                      
  Devuélvelo agrupado por capa, una línea por archivo, con este formato:              
  ruta/al/archivo — qué papel tiene en comentarios (rango de líneas relevante si el   
  archivo es compartido con otra feature)                                             
                                                                                      
  Reglas:                                                                             
  - Solo lista. No analices, no opines y no señales problemas todavía.                
  - No modifiques ni crees ningún archivo.                                            
  - Si un archivo es compartido (por ejemplo, el controlador de posts), inclúyelo e   
  indica qué líneas son de comentarios.                                               
  - Al final dime cuántos archivos son en total y si hay alguna capa donde no         
  encontraste nada.   
```

**Qué salió:** 
```
Total: 10 archivos. 1 de API, 3 de datos, 5 de web y 1 de test.

  Capas donde no encontré nada:
  - No hay ningún archivo de comentarios en estas capas: controlador, modelo Lucid
    (solo existe ComentarioSchema, que es generado), factory, validador, transformer,
    servicio y repositorio. En backend/app/ lo único que hay es de usuarios y
    autenticación.
  - En el frontend no hay ninguna llamada a PATCH ni a DELETE /comentarios/:id. Solo
    encontré la lista, la creación y el recuento.
```

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Audita SOLO la feature de comentarios de este repo (crear, editar, borrar y listar  
  comentarios), limitándote a los archivos del inventario que acabas de hacer. Si ves 
  problemas en posts, autores, categorías o etiquetas que no afecten a comentarios,   
  ignóralos y no los menciones.                                                       
                                                                                      
  El contraste son las reglas de negocio del README. Las que más afectan a esta       
  feature son:                                                                        
  - Regla 2: un comentario pendiente no se muestra en el detalle del post.            
  - Regla 5: el recuento de comentarios cuenta solo los aprobados.                    
  - Regla 6: la respuesta pública de un post no expone el correo del autor ni el de   
  quien comenta.                                                                      
  - Regla 1: un post en borrador no aparece en ningún sitio público (comprueba si se  
  puede comentar o ver comentarios de un borrador).                                   
                                                                                      
  Cada hallazgo debe incumplir una regla concreta (cítala por número) o producir un   
  fallo observable (crash, dato corrupto, acción no autorizada). Si solo es una       
  cuestión de estilo, descártalo.                                                     
                                                                                      
  Crea el archivo docs/auditoria/comentarios.md con una tabla en Markdown, una fila   
  por hallazgo, ordenada de mayor a menor prioridad, con exactamente estas columnas:  
                                                                                      
  | # | Qué es | Dónde está | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se     
  deshace con un revert? | ¿Bloquea o desbloquea a otro? |                            
                                                                                      
  Reglas de cada casilla:                                                             
  1. Qué es: una sola frase. Si necesitas más, son dos hallazgos: sepáralos en dos    
  filas.                                                                              
  2. Dónde está: ruta/archivo:línea más la línea de código citada literalmente entre  
  backticks. Si no puedes citar una línea real, NO escribas la fila.                  
  3. Qué se rompe y quién lo nota: el efecto concreto y a quién le llega (lector,     
  autor del post, quien comenta, administrador, o "nadie todavía").                   
  4. Sitios a tocar: un número exacto seguido de la lista de archivo:línea que habría 
  que cambiar. Contados, no estimados.                                                
  5. ¿Se deshace con un revert?: "Sí" o "No". Si es "No", explica por qué (datos ya   
  escritos en la base, migración aplicada, contrato de API que ya consume el          
  frontend…).                                                                         
  6. ¿Bloquea o desbloquea a otro?: el número de fila del otro hallazgo y en qué      
  sentido, o "A ninguno".                                                             
                                                                                      
  Reglas generales:                                                                   
  - Si no tienes evidencia para una casilla, déjala VACÍA. No la rellenes por         
  deducción ni por probabilidad.                                                      
  - No modifiques ningún otro archivo. No propongas ni apliques arreglos.             
  - No escribas nada debajo de la tabla: esa parte la escribo yo.                     
                                                                                      
  Cuando termines, dime solo: cuántos hallazgos escribiste y cuántas casillas dejaste 
  vacías.   
  ```
  
**Qué salió:** 
```
Escribí 14 hallazgos en docs/auditoria/comentarios.md y dejé 3 casillas vacías:
  - Fila 10: "¿Se deshace con un revert?" y "¿Bloquea o desbloquea a otro?".
  - Fila 11: "¿Bloquea o desbloquea a otro?".
```
