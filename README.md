# PROYECTO: VERO
# Sales call intelligence for growing teams

---

## CONTEXTO Y OBJETIVO

Construye Vero, una aplicación web SaaS B2B que graba llamadas de ventas, las transcribe con IA local, genera análisis automático de cada conversación y entrega coaching accionable al representante de ventas y al manager. El sistema debe funcionar completamente en local (sin enviar datos a APIs externas en su modo base) y estar diseñado para escalar a cloud en una segunda fase sin refactoring mayor.

---

## 01. IDENTIDAD DE MARCA

Nombre: Vero
Tagline: Sales call intelligence for growing teams
Tono de voz: Profesional, directo, sin exceso de tecnicismo. Habla como un analista senior, no como un bot.
Valores de diseño: Claridad sobre decoración. Densidad de información con elegancia. Sin elementos vacíos. Sin emojis en ninguna parte de la interfaz ni del código.
Audiencia primaria: Sales managers y reps en empresas de 5 a 150 personas que no pueden pagar Gong.

---

## 02. STACK TÉCNICO

Frontend:
- React 18 con TypeScript estricto
- Vite como bundler
- React Router v6 para navegación
- Zustand para estado global
- TanStack Query (React Query) para fetching y caché
- Tailwind CSS con design system propio (no usar componentes de terceros como shadcn sin adaptar al sistema visual de Vero)
- Recharts para visualizaciones de datos
- Web Audio API para captura de audio en el navegador

Backend:
- Node.js 20 con TypeScript
- Express.js con estructura modular
- Prisma ORM
- PostgreSQL como base de datos principal
- Redis para colas de jobs y caché de sesión
- BullMQ para procesamiento asíncrono de transcripciones y análisis
- Multer para upload de archivos de audio
- JWT para autenticación (access token 15min + refresh token 7 días)
- Helmet, CORS, rate-limiting, y express-validator en todas las rutas

IA Local (modo base):
- Whisper.cpp (modelo base o small, via llamada a proceso hijo) para transcripción
- Ollama con modelo llama3 o mistral para análisis y coaching
- Toda la inferencia corre en el servidor local, sin llamadas externas

Infraestructura de archivos:
- Audio almacenado localmente en /storage/audio con nombres hasheados
- Transcripciones en /storage/transcripts como JSON estructurado
- Logs de procesamiento en /storage/logs

---

## 03. ESTRUCTURA DE CARPETAS

/vero
├── /apps
│   ├── /web              (Frontend React)
│   │   ├── /src
│   │   │   ├── /assets           (fuentes, iconos SVG propios)
│   │   │   ├── /components
│   │   │   │   ├── /ui           (Button, Input, Badge, Card, Table, Modal, Toast — todos propios)
│   │   │   │   ├── /layout       (AppShell, Sidebar, TopBar, PageHeader)
│   │   │   │   └── /domain       (CallCard, ScoreRing, CoachingTip, TranscriptViewer, WaveformPlayer)
│   │   │   ├── /pages
│   │   │   │   ├── /auth         (Login, ForgotPassword)
│   │   │   │   ├── /dashboard    (Home con métricas globales)
│   │   │   │   ├── /calls        (Lista de llamadas, detalle de llamada)
│   │   │   │   ├── /coaching     (Vista de coaching por rep)
│   │   │   │   ├── /team         (Gestión de usuarios — solo para managers)
│   │   │   │   └── /settings     (Perfil, integraciones, modelo de IA)
│   │   │   ├── /hooks            (useRecorder, useCallAnalysis, useAuth, usePagination)
│   │   │   ├── /stores           (authStore, callsStore, uiStore)
│   │   │   ├── /services         (api.ts — cliente axios con interceptors)
│   │   │   ├── /types            (tipos compartidos frontend)
│   │   │   └── /utils            (formatters, validators, constants)
│   │   └── tailwind.config.ts    (con tokens de Vero)
│   │
│   └── /api              (Backend Node.js)
│       ├── /src
│       │   ├── /config           (env, database, redis, logger)
│       │   ├── /middleware        (auth, errorHandler, rateLimiter, validate, upload)
│       │   ├── /modules
│       │   │   ├── /auth          (controller, service, routes)
│       │   │   ├── /users         (controller, service, routes)
│       │   │   ├── /calls         (controller, service, routes)
│       │   │   ├── /transcription (service — worker BullMQ, Whisper bridge)
│       │   │   ├── /analysis      (service — Ollama bridge, prompts)
│       │   │   └── /coaching      (service — lógica de scores y tips)
│       │   ├── /jobs              (bullmq workers: transcribeJob, analyzeJob)
│       │   ├── /lib               (whisper.ts, ollama.ts, storage.ts)
│       │   ├── /types             (tipos compartidos backend)
│       │   └── app.ts, server.ts
│       └── /prisma
│           ├── schema.prisma
│           └── /migrations
├── /packages
│   └── /shared-types      (tipos compartidos entre web y api via monorepo)
├── docker-compose.yml      (postgres + redis en desarrollo local)
├── .env.example
└── README.md

---

## 04. SISTEMA DE DISEÑO (Vero Design System)

### Filosofía
Claridad sobre decoración. Sin gradientes. Sin sombras decorativas. Sin bordes redondeados excesivos. El espacio en blanco es el único ornamento válido. Cada elemento visual debe justificar su presencia.

### Tipografía
Fuente base: Inter (variable font via Google Fonts o self-hosted)
- Display: 28px / weight 500 / tracking -0.3px
- Heading 1: 22px / weight 500
- Heading 2: 18px / weight 500
- Body: 15px / weight 400 / line-height 1.65
- Small / label: 12px / weight 500 / tracking 0.04em / uppercase para etiquetas de categoría
- Monospace (transcripciones): JetBrains Mono 13px

### Paleta de color (tokens en tailwind.config.ts)
Primario: #1a1a1a (casi negro, no negro puro)
Superficie: #ffffff
Fondo: #f7f6f3 (warm off-white, no gris frío)
Borde: #e4e2dc
Borde hover: #c8c5bc
Acento: #0f62d4 (azul profundo, no brillante)
Acento hover: #0a4fa8
Éxito: #1a7a4a
Advertencia: #9a5c00
Error: #c0392b
Texto primario: #1a1a1a
Texto secundario: #6b6860
Texto terciario: #9b9890

Dark mode: implementar con clase 'dark' en el html root. Todos los tokens tienen su equivalente dark.

### Espaciado
Sistema de 4px base. Usar exclusivamente: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80px.

### Componentes UI obligatorios a construir (todos en /components/ui/)
- Button: variantes primary, secondary, ghost, destructive. Tamaños sm, md, lg. Estado loading con spinner.
- Input: con label flotante, mensaje de error, ícono izquierda/derecha opcional.
- Badge: variantes success, warning, error, neutral, info. Solo texto, sin íconos.
- Card: surface blanca, borde 1px, radius 8px. Variante clickable con hover state.
- Table: headers sticky, hover por fila, soporte para sorting por columna.
- Modal: overlay sutil (no negro), animación de entrada en 150ms, cierre con ESC.
- Toast: notificaciones en esquina inferior derecha, apilables, auto-dismiss en 4s.
- ScoreRing: SVG animado que muestra un score del 0 al 100 con color semántico.
- Skeleton: placeholders de carga para Card y Table.

### Iconos
Usar Lucide React. Tamaño estándar 16px en texto, 20px standalone. Sin íconos decorativos vacíos de significado.

### No usar
- No box-shadow en ningún componente de contenido. Solo en modales (muy sutil, 1 nivel).
- No gradientes en ningún caso.
- No animaciones de más de 250ms salvo que sean funcionales (loading, progreso).
- No emojis en ninguna parte de la interfaz.
- No texto en mayúsculas salvo en labels de categoría (Badge, TableHeader).

---

## 05. MÓDULOS DEL BACKEND

### Auth Module
- POST /api/auth/register — registro de organización + primer usuario (owner)
- POST /api/auth/login — devuelve access_token (15min) + refresh_token (7días, httpOnly cookie)
- POST /api/auth/refresh — rota tokens
- POST /api/auth/logout — invalida refresh token
- Middleware auth: verifica JWT, inyecta req.user con { id, orgId, role }
- Roles: OWNER, MANAGER, REP

### Users Module
- GET /api/users — lista usuarios de la org (solo MANAGER+)
- POST /api/users/invite — invita por email, genera token de activación
- PATCH /api/users/:id/role — cambia rol (solo OWNER)
- DELETE /api/users/:id — desactiva usuario (soft delete)

### Calls Module
- POST /api/calls/upload — recibe archivo .webm/.mp3/.mp4, lo guarda en storage, crea registro Call con status PENDING, encola job de transcripción. Máx 500MB.
- GET /api/calls — lista llamadas con filtros (rep, fecha, score, status). Paginación cursor-based.
- GET /api/calls/:id — detalle completo: metadata, transcripción, análisis, coaching tips.
- DELETE /api/calls/:id — elimina archivo y registro (soft delete).
- PATCH /api/calls/:id — actualiza metadata (nombre, notas, etiquetas).

### Transcription Service (BullMQ Worker)
- Recibe job con { callId, filePath }
- Llama a Whisper.cpp via child_process.spawn con el archivo de audio
- Parsea el output VTT/JSON de Whisper a estructura de segmentos: [{ start, end, speaker, text }]
- Guarda transcripción como JSON en /storage/transcripts/:callId.json
- Actualiza Call.status a TRANSCRIBED y llama al siguiente job de análisis
- Manejo de errores: si Whisper falla, Call.status = ERROR con mensaje descriptivo

### Analysis Service (BullMQ Worker + Ollama)
- Recibe transcripción completa
- Construye prompt estructurado para Ollama (ver sección 07)
- Recibe respuesta JSON del modelo con el análisis
- Calcula scores numéricos por categoría
- Guarda resultado en tabla CallAnalysis
- Actualiza Call.status = ANALYZED

### Coaching Service
- Genera tips accionables basados en el análisis
- Prioriza los 3 tips más relevantes para el rep
- Identifica patrones entre múltiples llamadas del mismo rep (comparativa histórica)

---

## 06. MÓDULOS DEL FRONTEND

### AppShell y navegación
Sidebar colapsable de 240px (expandido) / 56px (colapsado). Logo Vero en top-left. Navegación con íconos Lucide + texto. Indicador de rol activo en el bottom del sidebar. Sin hamburguesa en desktop.

### Dashboard (/)
- Métricas globales del equipo: llamadas esta semana, score promedio, rep con mejor y peor performance.
- Gráfico de línea: evolución del score promedio por semana (Recharts LineChart).
- Tabla de llamadas recientes con score, rep asignado, duración, estado.
- Todos los datos con skeleton loading mientras cargan.

### Calls List (/calls)
- Filtros: por rep (select), por rango de fechas (date picker simple), por score (slider), por status (badge seleccionable).
- Tabla con columnas: título, rep, fecha, duración, score, status, acciones.
- Ordenable por cualquier columna.
- Upload de nueva llamada: drag-and-drop area + botón. Progress bar durante upload y procesamiento.

### Call Detail (/calls/:id)
- Header: nombre de la llamada, rep, fecha, duración, score global como ScoreRing grande.
- Pestañas (sin librerías externas): Transcripción | Análisis | Coaching.
- Transcripción: texto segmentado por tiempo y speaker. Resaltado de momentos clave identificados por la IA. Audio player sincronizado con la transcripción (al hacer click en un segmento, el audio salta a ese momento).
- Análisis: scores por categoría (rapport, detección de necesidades, manejo de objeciones, cierre, escucha activa). Cada score con barra visual y explicación de 1–2 líneas.
- Coaching: lista de tips accionables priorizados. Cada tip con título, explicación, y cita exacta de la llamada que lo genera.

### Coaching View (/coaching)
- Vista por rep: selector de rep en el top. Muestra evolución de scores en el tiempo. Tendencias positivas y negativas. Últimos tips generados.

### Settings (/settings)
- Perfil: nombre, email, cambio de contraseña.
- Modelo de IA: selector del modelo Ollama instalado localmente (llama3, mistral, etc.). Test de conexión.
- Organización: nombre, logo (upload simple).
- Integraciones: placeholder visual para Zoom, Google Meet, HubSpot (no funcionales en v1, solo preparar la UI).

---

## 07. PIPELINE DE IA LOCAL

### Whisper Bridge (lib/whisper.ts)
- Detecta si whisper.cpp está instalado en el sistema (which whisper)
- Ejecuta: whisper [filepath] --output-json --language es --model base
- Timeout: 10 minutos. Si supera, mata el proceso y marca error.
- Parsea el JSON de salida a array de segmentos.
- Diarización básica: si el archivo tiene múltiples canales, intentar separar speakers por canal.

### Ollama Bridge (lib/ollama.ts)
- Cliente HTTP a localhost:11434 (port default de Ollama)
- Detecta si Ollama está activo al arrancar el servidor. Loguea advertencia si no.
- Función generateAnalysis(transcript: Segment[]): Promise
- Función generateCoaching(analysis: CallAnalysis, repHistory: CallAnalysis[]): Promise

### Prompt de análisis (en /modules/analysis/prompts.ts)
Construir el prompt como template function. El prompt debe:
1. Recibir la transcripción como texto plano con timestamps.
2. Pedir al modelo que responda ÚNICAMENTE en JSON válido, sin texto previo ni posterior.
3. Solicitar el siguiente esquema de respuesta:
{
  "summary": "string — resumen ejecutivo en 2–3 frases",
  "duration_seconds": number,
  "scores": {
    "rapport": { "score": 0-100, "evidence": "cita textual", "explanation": "string" },
    "needs_discovery": { "score": 0-100, "evidence": "cita textual", "explanation": "string" },
    "objection_handling": { "score": 0-100, "evidence": "cita textual", "explanation": "string" },
    "closing_attempt": { "score": 0-100, "evidence": "cita textual", "explanation": "string" },
    "active_listening": { "score": 0-100, "evidence": "cita textual", "explanation": "string" }
  },
  "talk_ratio": { "rep": number, "prospect": number },
  "key_moments": [{ "timestamp": number, "label": "string", "type": "positive|negative|neutral" }],
  "overall_score": number,
  "next_steps_mentioned": boolean
}
4. El prompt debe ser en español por defecto, con opción de inglés configurable.

---

## 08. SEGURIDAD

### Autenticación y sesiones
- Passwords: bcrypt con saltRounds 12. Nunca almacenar en texto plano.
- JWT access token firmado con HS256, expira en 15 minutos.
- Refresh token almacenado hasheado en base de datos. Un token válido por usuario. Rotación en cada uso.
- Refresh token en httpOnly, Secure, SameSite=Strict cookie. No en localStorage.
- Logout invalida el refresh token en base de datos.

### Autorización
- Cada request validado por middleware auth + middleware de rol.
- Scope de organización: TODAS las queries a base de datos incluyen WHERE orgId = req.user.orgId. Nunca exponer datos de otra organización.
- Los REPs solo ven sus propias llamadas a menos que el MANAGER los asigne explícitamente a otras.

### Validación de inputs
- express-validator en cada ruta que recibe datos. Nunca confiar en el cliente.
- Sanitización de strings antes de insertar en base de datos.
- Archivos de audio: validar MIME type real (no solo extensión) con file-type library. Solo permitir audio/webm, audio/mpeg, audio/mp4, video/mp4.
- Tamaño máximo de upload: configurable en .env, default 500MB.

### Headers y protección
- Helmet con configuración estricta de CSP.
- CORS: solo el dominio del frontend configurado en .env.
- Rate limiting: 100 requests/15min por IP en rutas generales. 5 intentos de login/min por IP.
- No exponer stack traces en responses de producción. Loguear internamente con Winston.

### Almacenamiento de archivos
- Archivos de audio guardados con nombre UUID + extensión, nunca el nombre original del archivo.
- Directorio storage fuera de la carpeta pública del servidor. No servibles directamente via URL.
- Audio servido via endpoint autenticado que verifica que el callId pertenece al usuario.

---

## 09. BASE DE DATOS (Prisma Schema)

Modelos requeridos:

Organization { id, name, slug, logoUrl, createdAt, settings (Json) }

User { id, orgId, name, email, passwordHash, role (OWNER|MANAGER|REP), isActive, lastLoginAt, createdAt }

RefreshToken { id, userId, tokenHash, expiresAt, createdAt }

Call { id, orgId, repId, title, notes, audioPath, audioDurationSeconds, status (PENDING|TRANSCRIBING|TRANSCRIBED|ANALYZING|ANALYZED|ERROR), errorMessage, createdAt, updatedAt }

CallTranscript { id, callId, segments (Json — array de segmentos), createdAt }

CallAnalysis { id, callId, summary, overallScore, scores (Json), talkRatio (Json), keyMoments (Json), nextStepsMentioned, rawModelOutput (Text), modelUsed, createdAt }

CoachingTip { id, callId, repId, priority (1-3), title, explanation, quote, category, createdAt }

Tag { id, orgId, name, color }
CallTag { callId, tagId }

Índices requeridos:
- Call: (orgId, status), (repId, createdAt), (orgId, createdAt)
- CoachingTip: (repId, createdAt)

---

## 10. CRITERIOS DE CALIDAD Y ENTREGA

### Criterios de construcción
- TypeScript estricto en frontend y backend. Cero any sin justificación documentada.
- Todos los errores manejados explícitamente. Sin .catch(() => {}) vacíos.
- Logging estructurado en el backend con Winston: nivel, timestamp, módulo, mensaje, contexto.
- Todas las variables de entorno en .env.example con descripción de cada una.
- Migraciones de base de datos versionadas. Nunca modificar una migración existente.
- Sin secrets hardcodeados en ningún archivo, incluyendo tests.

### Criterios de UI/UX
- Todo estado de loading tiene su skeleton o spinner correspondiente.
- Todo estado de error tiene su mensaje visible y accionable (no solo console.log).
- Todo estado vacío (sin llamadas, sin datos) tiene su empty state con mensaje y CTA.
- Los formularios muestran errores inline por campo, no solo un alert genérico.
- La aplicación es completamente funcional con teclado (Tab, Enter, Escape).
- Tiempo de respuesta percibido: usar optimistic UI donde sea posible.

### Criterios de performance
- Las listas de llamadas usan paginación cursor-based, nunca carga total.
- Las transcripciones largas se renderizan con virtualización (react-virtual).
- Los jobs de IA corren en workers separados y nunca bloquean el event loop del servidor HTTP.
- El análisis de una llamada de 30 minutos no debe afectar la disponibilidad de la API.

### Orden de construcción recomendado para el agente
1. Scaffold del monorepo, docker-compose, .env.example, Prisma schema y primera migración.
2. Backend: auth completo (register, login, refresh, logout) con tests.
3. Backend: calls module (upload, list, detail) sin IA.
4. Whisper bridge + job de transcripción funcionando con audio de prueba.
5. Ollama bridge + job de análisis funcionando.
6. Frontend: design system (todos los componentes UI base).
7. Frontend: autenticación (login, guards de ruta).
8. Frontend: dashboard y calls list.
9. Frontend: call detail con transcripción y análisis.
10. Frontend: coaching view y settings.
11. Revisión de seguridad completa.
12. README con instrucciones de instalación de Whisper y Ollama en local.

---

## INSTRUCCIÓN FINAL AL AGENTE

Construye cada módulo completo antes de pasar al siguiente. No dejes interfaces vacías, mocks, o TODOs sin resolver. Cada función debe funcionar o lanzar un error claro. El resultado final debe poder arrancarse con `docker-compose up` + `npm run dev` y mostrar una interfaz completamente operativa donde puedo subir un archivo de audio y recibir análisis en menos de 5 minutos en hardware moderno.

Cuando tengas una decisión de arquitectura que no esté cubierta aquí, elige siempre la opción más simple que funcione correctamente sobre la más sofisticada. El principio es: que funcione bien antes de que funcione bonito.
    
