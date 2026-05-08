---
tags: [sprint, setter, whatsapp, ai]
sprint: setter-whatsapp
duration: 3-4 weeks
status: active
created: 2026-05-08
start_date: 2026-05-08
end_date: 2026-05-31
target_mvp: 2026-05-11
---

# Sprint Setter WhatsApp — Listo Lunes 2026-05-11

Setter de WhatsApp **full optimizado para grupos y contactos**. Filtros para segmentar a quién hablar, carga de info e instrucciones, anti-banneo, handoff a humano cuando aplica, observability en vivo.

**MVP funcional para el lunes (3 días)**, optimización continua después.

## Métricas

- **25 tareas** en 7 sub-categorías
- **72.5 horas** total
- **Prioridad**: 11 high · 10 medium · 4 low
- **MVP lunes**: 8 tareas marcadas 🔴 = **19.5h ≈ 2.4 días-hombre**
- **Sprint UUID en DB**: `523b8b96-5d96-4200-94ce-639a4a0658c1`
- **Tenant**: black-wolf
- **URL**: `central.blackwolfsec.io/black-wolf/task-management` (sprint "Setter WhatsApp — Listo Lunes 2026-05-11")

## Distribución

| Sub-categoría | Tareas | Horas |
|---|---:|---:|
| `[setter-instrucciones]` | 5 | 18h |
| `[setter-anti-banneo]` | 5 | 10h |
| `[setter-handoff]` | 4 | 10h |
| `[setter-filtros]` | 5 | 11h (incluye uno medium en otra cat) |
| `[setter-observability]` | 3 | 16h |
| `[setter-grupos]` | 3 | 7h |
| `[setter-ux]` | 1 | 3h |

## Estado actual del setter (lo que YA existe)

Antes de meter las tareas auditamos el setter actual:

### Backend (`enjambre-api/`)

- `connectors/whatsapp.js`: setter integrado en el flujo de mensajes entrantes
- Config en `whatsapp_config`:
  - `setter_enabled` (bool global)
  - `setter_message` (texto plano único)
  - `setter_docs` (JSON, contiene `_profiles` array para multi-setter)
  - `setter_pipeline_id`, `setter_default_stage_key`
- Multi-setter: profiles JSON resueltos por `contactPipelineId`
  ```json
  { "profiles": [{ "name", "pipeline_id", "enabled", "delay_minutes",
                   "system_message", "docs", "audio_phrases" }] }
  ```
- `setter_docs/index.js`: carga `*.md` files concatenados como knowledge base global
- `agents/orchestrator.js > setterReply()`: modo ligero con prompt caching
- WhatsApp groups: parcialmente — `WA_WHITELIST_GROUPS` permite responder en grupos específicos, pero responde a TODO mensaje (sin filtro por mención)

### Frontend (`Dashboard-Ops-/`)

- `src/pages/ai-agents/AiSetterPage.jsx`: UI de configuración multi-perfil
- `src/pages/ai-agents/AiSetterMetrics.jsx`: dashboard de métricas
- Filtros UI hoy: solo `pipeline` y `status`

### Lo que falta (core del sprint)

1. **Filtros más finos**: stage_key, tags, custom_fields, owner_scope, ventana horaria, blacklist
2. **Knowledge base estructurada**: FAQs, objeciones, productos, scripts (no solo `.md` plano)
3. **System prompt con variables**: `{{nombre}}, {{stage}}, {{producto_pagado}}` con sustitución real
4. **Anti-banneo robusto**: typing indicator, rate limit por número, variación de mensajes
5. **Comportamiento en grupos**: solo responder si lo mencionan, comandos admin
6. **Handoff a humano**: detectar intent de compra / queja, escalar con notif Telegram
7. **Observability en vivo**: logs por sesión, métricas dashboard, dry-run
8. **A/B testing y versionado** (futuro)

---

## 🔴 MVP para Lunes 2026-05-11 (19.5h · ~2.4 días-hombre)

8 tareas bloqueantes que dejan el setter "listo y operativo" para el lunes:

### Instrucciones (8h)

- **System prompt editable per-profile con variables** (4h) — parser `{{nombre}}, {{stage_actual}}, {{ultimo_producto_pagado}}, {{owner_scope}}`. Frontend: textarea + preview con contacto de ejemplo.
- **Knowledge base por cliente** (4h) — `whatsapp_config.setter_docs.knowledge_base = { faqs, objeciones, productos, scripts }`. UI: editor estructurado tab Knowledge.

### Filtros (6.5h)

- **Ampliar filtros UI** (3h) — stage_key, tags (jsonb intersect), custom_fields (key=valor), owner_scope. Backend: `setter_resolveProfile` extendido.
- **Ventana horaria** (2h) — `setter_active_hours = { monday: { start: 9, end: 20 }, ... }` en TZ del cliente. Skip si fuera.
- **Black-list de contactos** (1.5h) — `custom_fields.setter_blacklist=true` o tabla `setter_exclusions`. Botón "Excluir del setter" en CRM.

### Grupos (4h)

- **Auto-detección de menciones** (2h) — bot responde solo si `message.body` incluye `@<nombre_bot>` o `mentionedIds` incluye el number. Default OFF en grupos nuevos.
- **Comandos `/setter pause|resume|status|reload`** (2h) — admins controlan desde el grupo sin restart.

### Anti-banneo (1h)

- **Typing indicator + delay random** (1h) — `chat.sendStateTyping()` → wait random(2-6s) → `clearState()` → enviar.

---

## 🟠 Post-MVP (semana 12-31 may)

10 tareas medium — 27h totales.

- Rate limit por número (max 10/h por contacto)
- Stop conditions: escalar a humano si N msgs sin respuesta
- Logs por sesión visibles en frontend
- Detección intent de compra → tag + ping closer
- Detección queja/cancelación → escalar humano
- Métricas en vivo (msgs/h, response time, cost/día, conversion)
- Template del primer mensaje per-stage
- Audio phrases con uploads
- Variación natural de mensajes
- Filtro de palabras prohibidas pre-envío

## 🟡 Optimización continua

4 tareas low — 17h.

- Respuesta contextual en grupos (lee hilo)
- Versionado de system prompts
- A/B testing de variantes
- Rotación de números (evitar quemar uno solo)

---

## Decisiones de producto pendientes

Antes de lunes hay que cerrar:

1. **¿En qué cliente probamos primero?** Recomendación: **asesorias-suiza Portillo** (Stripe live, multi-owner ya soportado, KPIs claros).
2. **¿Multi-perfil o un solo setter por cliente al arrancar?** Sugiero arrancar con 1 perfil per-pipeline en el cliente piloto y escalar después.
3. **¿Qué knowledge base inicial?** Hay que pedirle al cliente:
   - Lista de FAQs típicas (preguntas que reciben en WhatsApp)
   - Top 5 objeciones (precio, tiempo, dudas técnicas) y respuesta ideal
   - Productos / precios
   - Tonos de voz (formal vs cercano)
   - Lo que NO debe decir el setter (palabras prohibidas, promesas no respaldables)
4. **Ventana horaria default**: 9-20 lunes a viernes? ¿Domingos off?
5. **Notif Telegram a quién**: Alejandro, owner del cliente, ambos?

## Conexiones

- Estado backend: [[Enjambre-API]] (connector whatsapp.js)
- Plataforma: [[Modulo-AI-Agents]]
- Cliente piloto: [[AsesoriaSuiza-Portillo-Lukas]]
- Infraestructura cognitiva: [[Donna-y-Army]]
- Riesgo de baseline: [[WhatsApp]] (banneo 26-abr Creator Founder)
- Sprint hermano: [[Sprint-Arreglos]] (deuda general)
