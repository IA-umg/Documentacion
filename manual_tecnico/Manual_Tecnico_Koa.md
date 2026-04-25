# UNIVERSIDAD MARIANO GÁLVEZ DE GUATEMALA
**FACULTAD DE INGENIERÍA EN SISTEMAS DE INFORMACIÓN**  
**LICENCIATURA EN INGENIERÍA EN SISTEMAS DE INFORMACIÓN**

**NOVENO CICLO**  
Inteligencia Artificial

---

# Koa — Manual de Técnico

**Estudiantes:**
- Wilson Adolfo Coc Avila — 1990-22-11648
- Daniel Angel Ambrocio Coj — 1990-22-13443
- Amanda Sofia Mejia Paniagua — 1990-22-19646

---

## Contenido

1. [Introducción](#introducción)
2. [Objetivo del Sistema](#objetivo-del-sistema)
3. [Alcance](#alcance)
4. [Reporte de Evaluación MVP - RAG Académico](#reporte-de-evaluación-mvp---rag-académico)
   - [Cumplimiento de Requisitos Base](#cumplimiento-de-requisitos-base)
   - [Características Avanzadas (Implementadas)](#características-avanzadas-implementadas)
   - [Experimentos Obligatorios](#experimentos-obligatorios)
5. [Arquitectura del Sistema](#arquitectura-del-sistema)
6. [Flujo del Sistema](#flujo-del-sistema)
7. [Tecnologías Utilizadas](#tecnologías-utilizadas)
8. [Estructura del Proyecto](#estructura-del-proyecto)
9. [Instalación del Sistema](#instalación-del-sistema)
   - [Clonación del repositorio](#clonación-del-repositorio)
   - [Instalación de dependencias](#instalación-de-dependencias)
   - [Variables de entorno](#variables-de-entorno)
   - [Ejecución del sistema](#ejecución-del-sistema)
10. [Configuración del Sistema](#configuración-del-sistema)
11. [API y Endpoints](#api-y-endpoints)
12. [Integración con Inteligencia Artificial](#integración-con-inteligencia-artificial)
13. [Despliegue](#despliegue)
14. [Mantenimiento](#mantenimiento)

---

## Introducción

El presente documento describe el desarrollo, arquitectura, instalación y funcionamiento del sistema backend del proyecto **"backend_IA"**. Este sistema forma parte de una solución orientada a inteligencia artificial, diseñada para procesar datos, gestionar lógica de negocio y proporcionar servicios a clientes externos (frontend u otros sistemas).

El backend constituye la capa central del sistema, encargada de la comunicación con servicios de IA, procesamiento de solicitudes y gestión de recursos.

---

## Objetivo del Sistema

Desarrollar un backend robusto que permita:

- Procesar solicitudes relacionadas con inteligencia artificial
- Gestionar lógica de negocio
- Servir como intermediario entre usuario y modelo de IA
- Garantizar escalabilidad y mantenimiento

El uso de repositorios en plataformas como GitHub permite colaboración, control de versiones y trazabilidad del desarrollo.

---

## Alcance

**El sistema incluye:**

- API backend
- Integración con servicios de IA
- Manejo de datos
- Configuración del entorno
- Despliegue básico

**No incluye:**

- Frontend (interfaz gráfica)
- Infraestructura avanzada (Kubernetes, cloud completo)

---

## Reporte de Evaluación MVP - RAG Académico

### Cumplimiento de Requisitos Base

#### 1.1 Ingesta de Documentos y Base de Datos

El sistema soporta la carga masiva y concurrente de archivos (`.pdf`, `.docx`, `.txt`, `.md`). Está diseñado para escalar desde decenas hasta miles de documentos, extrayendo el contenido y conservando metadatos clave para trazabilidad.

#### 1.2 Pipeline RAG Core

El pipeline implementado en el backend Node.js incluye:

- **Chunking:** Particionado dinámico del texto de los documentos para mantener contexto semántico.
- **Embeddings:** Vectorización a través del modelo principal configurado (`text-embedding-001`).
- **Indexación:** Almacenamiento optimizado utilizando **PostgreSQL + pgvector**. Se emplean índices de alta eficiencia como **HNSW** o **IVFFlat**. El sistema incluye un **mecanismo de fallback automatizado**: si la extensión pgvector no está disponible o falla, el backend detecta el error y ejecuta una operación manual de búsqueda degradada para evitar que el sistema colapse.

#### 1.3 Interfaz de Usuario (UI)

Se desarrolló una interfaz web moderna que permite:

- Subir archivos mediante drag-and-drop.
- Seleccionar el entorno de trabajo (Localhost o Producción).
- Configurar metadatos pre-ingesta (Curso, Materia).
- Visualizar las estadísticas del sistema de vectores en tiempo real.

#### 1.4 Endpoints y APIs

La API de consulta principal retorna una respuesta completa basada en **Streaming SSE**, incluyendo:

- **Texto Generado:** Formateado en Markdown y servido en chunks progresivos.
- **Fragmentos Usados:** Array con el contexto exacto inyectado al LLM.
- **Fuentes:** Metadatos auditables y citas en la propia respuesta del chat.

#### 1.5 Stack Tecnológico y Librerías Principales

- **Backend:** Node.js (API), `@langchain/community` (orquestación), `pdf-parse` (extracción de texto), SDKs de Gemini.
- **Base de Datos:** PostgreSQL con la extensión pgvector (Neon / Railway).
- **Frontend:** Next.js (React), `@tanstack/react-query` (estado asíncrono), `react-markdown` y `remark-gfm` (renderizado de respuestas e interfaces ricas).

---

### Características Avanzadas (Implementadas)

#### A. Respuesta "No sé" si no hay evidencia (Faithfulness)

Se configuró un estricto *System Prompt* que prohíbe al LLM inventar información. Si el recuperador no devuelve fragmentos relevantes, el modelo indicará naturalmente que carece de la información solicitada en los documentos base, erradicando las alucinaciones.

#### B. Filtros por Metadata

El pipeline soporta el particionado por metadatos (curso, materia). Esto permite realizar un pre-filtrado lógico antes de la búsqueda vectorial, limitando el espacio de búsqueda únicamente a los documentos relevantes para el dominio académico consultado.

#### 3. Evaluación y Calidad (Métricas)

**3.1 Calidad del Retrieval**

- **Precision@k:** Alta. Gracias al filtrado por metadata, el ruido de otras materias no contamina los resultados.
- **Recall@k:** Adecuada. Los embeddings de alta dimensionalidad logran agrupar relaciones semánticas indirectas.

**3.2 Calidad de Respuesta**

- **Faithfulness:** Excelente. Validado bloqueando fugas de "Chain of Thought" en la respuesta.
- **Uso de fuentes:** Las citas se imprimen directamente como `[1]`, correspondientes uno-a-uno con el documento fuente, presentados en los tooltips interactivos de la interfaz.

**3.3 Ingeniería y Producción**

- **Latencia:** Minimizada de cara al usuario gracias a la arquitectura Server-Sent Events (SSE).
- **Costo estimado:** Controlado al acotar el `topK` y no sobrecargar el prompt context window con documentos irrelevantes.
- **Manejo de Errores:** Fallbacks automatizados tanto a nivel backend (conexiones de base de datos) como frontend (timeouts y respuestas degradadas).

---

### Experimentos Obligatorios

#### Experimento 1: Tamaño del Chunk

- **Variaciones:** 500 tokens vs 900 tokens (overlap 120).
- **Resultados:** Los chunks de 500 tokens frecuentemente cortaban conceptos a la mitad, causando que el LLM no tuviera contexto suficiente para responder de forma completa. Los de 900 retuvieron hilos lógicos enteros de documentos técnicos (como ejercicios de matemáticas).
- **Decisión Final:** Se fijó el tamaño de chunk en **900** con **120** de superposición para capturar el contexto completo y mejorar el *Recall*.

#### Experimento 2: Valor de K (Resultados)

- **Variaciones:** k=2 vs k=5.
- **Resultados:** k=2 causaba bajo *Recall* en preguntas elaboradas. k=5 aumentaba levemente la latencia y costo de input pero garantizaba suficiencia de contexto.
- **Decisión Final:** Punto intermedio de **topK=4** para balancear costo de LLM API vs contexto aportado.

#### Experimento 3: Modelos de Embedding

- **Variaciones:** Gemini `text-embedding-001` vs `text-embedding-002`.
- **Resultados:** Ambos modelos tienen latencia y costos similares. Sin embargo, `text-embedding-002` mostró una leve mejoría en capturar la intención semántica de preguntas formuladas de manera compleja.
- **Decisión Final:** Se validó la viabilidad técnica con la familia de embeddings de Gemini (001 y 002) confirmando excelente calidad de Retrieval para textos académicos en español, utilizando 001 en despliegues iniciales por cuota.

---

## Arquitectura del Sistema

El sistema sigue una arquitectura por capas (posiblemente MVC o similar), lo cual facilita la separación de responsabilidades.

### 4.1 Arquitectura General

Capas principales:

- **Capa de Presentación (API)**
  - Endpoints REST
  - Recepción de solicitudes
- **Capa de Lógica de Negocio**
  - Procesamiento de datos
  - Validaciones
  - Integración con IA
- **Capa de Datos**
  - Acceso a base de datos o almacenamiento

---

## Flujo del Sistema

1. Cliente envía petición HTTP
2. API recibe la solicitud
3. Se procesa en la lógica de negocio
4. Se consulta modelo de IA (si aplica)
5. Se devuelve respuesta al cliente

---

## Tecnologías Utilizadas

| Componente | Tecnología |
|---|---|
| Lenguaje | JavaScript |
| Framework backend | Express.js / FastAPI |
| Control de versiones | Git |
| Plataforma | GitHub |
| IA | Modelos de lenguaje o APIs externas |

---

## Estructura del Proyecto

```
backend_IA/
│
├── src/
│   ├── controllers/
│   ├── services/
│   ├── models/
│   ├── routes/
│   └── config/
│
├── package.json / requirements.txt
├── README.md
└── .env
```

**Descripción de carpetas:**

| Carpeta | Descripción |
|---|---|
| `controllers/` | Manejo de peticiones HTTP |
| `services/` | Lógica de negocio |
| `models/` | Estructura de datos |
| `routes/` | Definición de endpoints |
| `config/` | Configuración del sistema |

---

## Instalación del Sistema

### Requisitos

- Node.js
- Git
- Editor (VS Code recomendado)

### Clonación del repositorio

```bash
git clone https://github.com/IA-umg/backend_IA.git
cd backend_IA
```

### Instalación de dependencias

```bash
npm install
```

### Variables de entorno

Crear archivo `.env`:

```env
PORT=3000
API_KEY=tu_clave
DB_URL=conexión
```

### Ejecución del sistema

```bash
npm run dev
```

---

## Configuración del Sistema

El sistema permite configurar:

- Puerto del servidor
- Claves de API
- Conexión a base de datos
- Parámetros del modelo de IA

---

## API y Endpoints

| Método | Endpoint | Descripción |
|---|---|---|
| GET | `/api/status` | Estado del sistema |
| POST | `/api/ia` | Procesa solicitud IA |
| POST | `/api/data` | Guarda información |

---

## Integración con Inteligencia Artificial

El sistema puede integrar modelos de IA mediante APIs o servicios externos. Plataformas modernas permiten probar y evaluar modelos directamente en repositorios, facilitando el desarrollo.

**Funciones principales:**

- Procesamiento de texto
- Generación de respuestas
- Análisis de datos

---

## Despliegue

El sistema puede desplegarse en:

- Servidor local
- VPS
- Servicios cloud (AWS, Azure, etc.)

**Pasos básicos:**

1. Configurar entorno
2. Instalar dependencias
3. Ejecutar servidor
4. Abrir puerto

---

## Mantenimiento

Se recomienda:

- Actualizar dependencias
- Revisar logs
- Controlar errores
- Versionar cambios
