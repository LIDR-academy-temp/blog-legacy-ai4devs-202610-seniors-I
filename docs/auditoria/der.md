# DER de la base de datos (SQLite)

Sacado de las migraciones en `backend/database/migrations/`. Línea **punteada** = relación
lógica que **no existe como clave foránea** en la base. Línea **continua** = FK real.

```mermaid
erDiagram
    autores {
        int id PK
        string nombre
        string email
        string rol "nullable"
        text bio "nullable"
        string avatar "nullable"
        text redes "JSON como texto"
        datetime created_at
        datetime updated_at
    }

    categorias {
        int id PK
        string nombre
        string slug "sin unique"
        datetime created_at
        datetime updated_at
    }

    etiquetas {
        int id PK
        string nombre
        string slug
        datetime created_at
        datetime updated_at
    }

    posts {
        int id PK
        string titulo
        string slug "sin unique"
        text resumen
        text cuerpo
        string imagen_portada
        string estado "borrador | publicado"
        datetime publicado_en
        int autor_id "sin FK"
        int categoria_id "sin FK"
        string autor_nombre "copia de autores"
        string autor_email "copia de autores"
        string categoria_nombre "copia de categorias"
        string etiquetas "CSV: 'a, b, c'"
        datetime created_at
        datetime updated_at
    }

    comentarios {
        int id PK
        int post_id "sin FK ni indice"
        string autor_nombre
        string autor_email
        text cuerpo
        string estado "texto libre: pendiente | aprobado | Aprobado | ok"
        datetime creado_en
        int padre_id "sin FK, autorreferencia"
    }

    users {
        int id PK
        string full_name
        string email UK
        string password
        timestamp created_at
        timestamp updated_at
    }

    auth_access_tokens {
        int id PK
        int tokenable_id FK
        string type
        string name
        string hash
        text abilities
        timestamp created_at
        timestamp updated_at
        timestamp last_used_at
        timestamp expires_at
    }

    autores    ||..o{ posts       : "autor_id (logica)"
    categorias ||..o{ posts       : "categoria_id (logica)"
    posts      ||..o{ comentarios : "post_id (logica)"
    comentarios |o..o{ comentarios : "padre_id (respuestas)"
    etiquetas  }o..o{ posts       : "posts.etiquetas = CSV, sin tabla pivote"
    users      ||--o{ auth_access_tokens : "tokenable_id (FK real, CASCADE)"
```

## Cómo leerlo

- **Bloque del blog** (`autores`, `categorias`, `etiquetas`, `posts`, `comentarios`): ninguna
  relación tiene FK. Todas las uniones dependen de que la aplicación mantenga los ids bien.
- **`posts` guarda copias** de datos de otras tablas (`autor_nombre`, `autor_email`,
  `categoria_nombre`), que pueden quedar desactualizadas.
- **`etiquetas` ↔ `posts`** es N:M, pero se resolvió con una cadena separada por comas en
  `posts.etiquetas` en lugar de una tabla pivote.
- **`comentarios`** se une a su post por `post_id` y a su comentario padre por `padre_id`
  (respuestas de un nivel), ambos sin FK ni índice. Ver hallazgos #8 y #13 de
  [`comentarios.md`](comentarios.md).
- **Bloque de autenticación** (`users`, `auth_access_tokens`): es el único con FK real, y no
  tiene relación con el blog. Un autor del blog (`autores`) no es un `user`.
