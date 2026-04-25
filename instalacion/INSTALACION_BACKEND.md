# Instalación — Backend RAG

## Requisitos previos

- [Node.js](https://nodejs.org/) >= 20.x (LTS recomendado)
- [Git](https://git-scm.com/)
- PostgreSQL con extensión [`pgvector`](https://github.com/pgvector/pgvector) — se usa [Neon](https://neon.tech) (plan gratuito)

**APIs externas:**

| Servicio | Uso | Dónde obtener la clave |
|---|---|---|
| Google AI Studio | Embeddings + LLM Gemini | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |
| Groq Console | LLM Llama (opcional) | [console.groq.com/keys](https://console.groq.com/keys) |

---

## 1. Clonar el repositorio

```bash
git clone https://github.com/IA-umg/backend_IA.git
cd backend_IA
```

---

## 2. Instalar dependencias

```bash
npm install
```

---

## 3. Configurar la base de datos (Neon + pgvector)

### 3.1 Crear proyecto en Neon

1. Crear una cuenta en [neon.tech](https://neon.tech).
2. Crear un nuevo **proyecto** y una **base de datos**.
3. Copiar la **cadena de conexión** desde el dashboard:

```
postgresql://usuario:password@host.neon.tech/nombre_bd?sslmode=require
```

### 3.2 Ejecutar la migración inicial

En el **SQL Editor** de Neon, copiar y ejecutar el contenido del archivo:

```
sql/migrations/001_schema_inicial.sql
```

Este script crea:

| Recurso | Descripción |
|---|---|
| Extensión `vector` | Habilita pgvector (tipos `vector` y `halfvec`) |
| Tabla `usuario` | Usuarios con autenticación (nombre, email, password_hash) |
| Tabla `documentos_rag` | Fragmentos indexados con embedding vectorial y metadata JSONB |
| Índice GIN | Filtros sobre `metadata` con operador `@>` |
| Índice HNSW | Búsqueda vectorial por similitud coseno sobre `embedding_vector` |

> El script es idempotente (`IF NOT EXISTS`), se puede ejecutar sin riesgo si las tablas ya existen.

---

## 4. Variables de entorno

Copiar el archivo de ejemplo y completar con valores reales:

```bash
cp .env.example .env
```

### Configuración completa del `.env`

```env
# ─── Servidor ────────────────────────────────────────────────────────────────
PORT=3000
NODE_ENV=development

# ─── Base de datos (Neon) ────────────────────────────────────────────────────
NEON_DATABASE_URL=postgresql://usuario:password@host.neon.tech/nombre_bd?sslmode=require

# ─── Google Gemini ───────────────────────────────────────────────────────────
GOOGLE_API_KEY=tu_clave_de_google_ai_studio
EMBEDDING_MODEL=gemini-embedding-001
EMBEDDING_DIMENSIONS=3072
LLM_MODEL=gemini-2.5-flash-lite

# ─── Groq (Llama) ────────────────────────────────────────────────────────────
GROQ_API_KEY=tu_clave_de_groq
GROQ_MODEL=llama-3.3-70b-versatile

# ─── Autenticación JWT ───────────────────────────────────────────────────────
JWT_SECRET=cadena_secreta_aleatoria_de_al_menos_32_caracteres
JWT_EXPIRES_IN=7d

# ─── CORS ────────────────────────────────────────────────────────────────────
CORS_ORIGIN=http://localhost:5173
FRONTEND_URL=http://localhost:5173

# ─── Configuración RAG ───────────────────────────────────────────────────────
RAG_CHUNK_SIZE=900
RAG_CHUNK_OVERLAP=120
RAG_TOP_K=4
RAG_TOP_K_DOCUMENTOS=2
RAG_TOP_K_REGISTROS=2

# ─── Fuente de registros de BD (opcional) ────────────────────────────────────
RAG_RECORDS_TABLE=
RAG_RECORDS_ID_COLUMN=id
RAG_RECORDS_TEXT_COLUMN=contenido
RAG_RECORDS_METADATA_COLUMN=metadata
RAG_RECORDS_SOURCE_LABEL=registros

# ─── Subida de archivos ──────────────────────────────────────────────────────
MAX_UPLOAD_BYTES=10485760
```

### Referencia rápida

| Variable | Obligatoria | Descripción |
|---|---|---|
| `NEON_DATABASE_URL` | ✅ | Cadena de conexión PostgreSQL (Neon) |
| `GOOGLE_API_KEY` | ✅ | Clave API de Google AI Studio |
| `JWT_SECRET` | ✅ | Secreto para firmar tokens JWT |
| `GROQ_API_KEY` | ⚠️ Recomendada | Clave para usar Llama vía Groq |
| `CORS_ORIGIN` | ✅ | URL del frontend permitido |
| `EMBEDDING_DIMENSIONS` | ✅ | Debe coincidir con la columna en BD (`3072`) |
| `RAG_RECORDS_TABLE` | ❌ Opcional | Solo si se quiere consultar una tabla adicional |

---

## 5. Ejecutar el servidor

### Desarrollo (con recarga automática)

```bash
npm run dev
```

### Producción

```bash
npm start
```

El servidor estará disponible en `http://localhost:3000` (o el puerto configurado en `PORT`).

---

## 6. Estructura del proyecto

```
backend_IA/
├── sql/
│   └── migrations/
│       ├── 001_schema_inicial.sql           ← Esquema completo
│       ├── 004_documentos_rag_metadata.sql  ← Historial: columna metadata
│       ├── 005_enable_pgvector.sql          ← Historial: extensión pgvector
│       └── 006_documentos_rag_embedding_vector.sql ← Historial: columna vectorial
├── src/
│   ├── config/
│   │   └── env.js               ← Variables de entorno
│   ├── db/
│   │   └── neon.js              ← Pool de conexión PostgreSQL
│   ├── routes/
│   │   ├── auth.routes.js       ← Endpoints de autenticación
│   │   ├── health.routes.js     ← Health check
│   │   └── rag.routes.js        ← Endpoints RAG e ingesta
│   ├── services/
│   │   ├── auth.service.js      ← Lógica de registro y login
│   │   ├── embedding.service.js ← Generación de embeddings (Gemini)
│   │   ├── file-ingest.service.js ← Procesamiento de archivos
│   │   └── rag.service.js       ← Núcleo RAG: retrieval + generación
│   ├── utils/
│   │   └── helpers.js           ← Utilidades compartidas
│   ├── app.js                   ← Configuración de Hono (middlewares, rutas)
│   └── index.js                 ← Punto de entrada del servidor
├── .env.example                 ← Plantilla de variables de entorno
├── .npmrc                       ← Configuración de npm (peer deps)
├── package.json
└── README.md
```

---

## 7. Notas técnicas

- **halfvec(3072):** Se usa media precisión para ahorrar almacenamiento con embeddings de 3072 dimensiones. Si se cambia `EMBEDDING_DIMENSIONS`, es necesario recrear la columna `embedding_vector`.
- **Índice HNSW:** Búsqueda aproximada de vecinos cercanos con similitud coseno. Si `pgvector` no está disponible, el sistema cae automáticamente a búsqueda legacy (coseno calculado en memoria).
- **Auto-schema:** El servicio RAG verifica y crea las columnas vectoriales automáticamente al arrancar.
- **Streaming SSE:** El endpoint `/rag/preguntar/stream` envía la respuesta token a token usando Server-Sent Events.
