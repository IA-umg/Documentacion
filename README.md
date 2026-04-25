# Koa — Asistente Inteligente con RAG Académico

> Asistente de inteligencia artificial basado en RAG (Retrieval-Augmented Generation) orientado a consultas académicas sobre SQL y documentos técnicos.

---

## Descripción

Koa es un sistema de IA conversacional desarrollado como proyecto universitario en la **Universidad Mariano Gálvez de Guatemala**, Facultad de Ingeniería en Sistemas de Información. Permite a los usuarios realizar consultas en lenguaje natural sobre una base de conocimientos alimentada con documentos propios, obteniendo respuestas precisas y con citas de fuente.

---

## Demo

[https://koa-app-nine.vercel.app/](https://koa-app-nine.vercel.app/)

---

## Repositorios

| Componente | Repositorio |
|---|---|
| Backend | `https://github.com/IA-umg/backend_IA` |
| Frontend | `https://github.com/IA-umg/koa-app` |

---

## Funcionalidades

- Chat conversacional con IA (modelos Gemini y Groq)
- Carga de documentos (`.pdf`, `.docx`, `.txt`, `.md`) para alimentar la base de conocimientos
- Búsqueda semántica mediante embeddings y pgvector
- Respuestas con citas y fuentes auditables
- Autenticación de usuarios (registro e inicio de sesión)
- Streaming de respuestas en tiempo real via SSE

---

## Arquitectura

```
Usuario → Frontend (Next.js)
              ↓
         Backend API (Node.js / Express)
              ↓
     ┌────────────────────┐
     │  Pipeline RAG      │
     │  - Chunking        │
     │  - Embeddings      │
     │  - pgvector Search │
     └────────────────────┘
              ↓
         LLM (Gemini / Grok)
              ↓
         Respuesta con fuentes
```

---

## Stack Tecnológico

### Backend
- **Node.js** + Express.js
- **LangChain** (`@langchain/community`) — orquestación del pipeline RAG
- **PostgreSQL** + **pgvector** — almacenamiento y búsqueda vectorial
- **Gemini SDK** — embeddings y generación de respuestas
- `pdf-parse` — extracción de texto desde PDFs

### Frontend
- **Next.js** (React)
- **Bun** — runtime y gestor de paquetes
- `@tanstack/react-query` — manejo de estado asíncrono
- `Axios` — manejo de peticiones
- `shadcn` — componentes
- `react-markdown` + `remark-gfm` — renderizado de respuestas

### Infraestructura
- **Vercel** — despliegue del frontend
- **Neon** — base de datos PostgreSQL en la nube
- **Railway** — despliegue del backend

---

## Autores

| Nombre | Carné |
|---|---|
| Wilson Adolfo Coc Avila | 1990-22-11648 |
| Daniel Angel Ambrocio Coj | 1990-22-13443 |
| Amanda Sofia Mejia Paniagua | 1990-22-19646 |

---

## Información académica

- **Universidad:** Universidad Mariano Gálvez de Guatemala
- **Facultad:** Ingeniería en Sistemas de Información
- **Carrera:** Licenciatura en Ingeniería en Sistemas de Información
- **Ciclo:** Noveno
- **Curso:** Inteligencia Artificial