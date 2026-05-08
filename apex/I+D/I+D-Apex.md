---
tags: [apex, i+d, investigacion, roadmap-tecnico]
created: 2026-05-08
updated: 2026-05-08
---

# I+D — Apex

Investigación y desarrollo abiertos sobre la línea Apex (Dashboard-Ops + Enjambre-API + capa cognitiva). Aquí viven las apuestas técnicas que aún no son producción: spikes, prototipos, hipótesis a validar, tech radar.

> Distinto de [[_moc/MOC-Roadmap]] (eso es trabajo decidido y planificado). Aquí vive lo que **estamos pensando** o **probando** sin compromiso firme.

## Áreas activas

### 1. Migración a RLS real (away from allow-all)

**Hipótesis**: 127 tablas con políticas `ALLOW ALL` se pueden migrar a RLS estricta sin romper Dashboard-Ops si el frontend pasa siempre `client_id` en el JWT y el backend usa rol autenticado en vez de `service_role` para queries del usuario.

**Estado**: research. Sin prototipo todavía. Hay que mapear qué queries actuales rompen si se aplica RLS estricta — probablemente el 30% de los reads del CRM (los que no filtran por `client_id` confiando en el frontend).

**Riesgo si NO se hace**: cualquier alumno con anon key puede leer datos de otros tenants. Hoy es solo enforced en frontend → trivial bypass con `curl + anon key`. Ver [[Bugs-Criticos#C6]].

**Spike sugerido**: `feat/rls-spike-formacion` — convertir las 8 tablas del módulo Formación a RLS estricta primero. Si funciona end-to-end, replicar al resto módulo a módulo.

**Quién**: TBD. Requiere alguien que entienda Postgres + Supabase auth profundo.

---

### 2. WhatsApp Business API (replace whatsapp-web.js)

**Hipótesis**: `whatsapp-web.js` es frágil (banneos cíclicos, sesiones que se caen al reiniciar, sin SLA oficial). Migrar a WhatsApp Business API daría estabilidad pero a coste de UX (templates aprobados, no se puede iniciar conversación libre).

**Estado**: research. Decisión pendiente sobre si vale la pena el coste de UX para la estabilidad.

**Trigger del go-decision**: cuando el siguiente banneo de un número productivo (Hugo, Asesoría Suiza) cause downtime de >24h en CRM-WhatsApp. Hoy llevamos un banneo cada 2-3 semanas (ver [[Creator-Founder]] — banneo 26-abr-2026).

**Spike sugerido**: contratar 1 número Cloud API de prueba con Meta (gratis primeros 1000 msgs/mes) y replicar el flujo CRM con webhooks templates-only. Validar que el [[Modulo-CRM]] sigue siendo usable.

**Open**: si lo hacemos, ¿migrar todos los tenants de golpe o por cohort? ¿Quién absorbe el coste de los templates aprobados (BlackWolf o el cliente)?

---

### 3. Hotmart connector

**Hipótesis**: Hugo y futuros tenants infoproducto venden también por Hotmart. Hoy esas ventas NO entran al CRM → datos parciales en pipelines.

**Estado**: scaffolding sin construir. Memoria del proyecto lo marca como "para Hugo" pero el conector no existe.

**Lo que hay**: nada en código. Lo que se sabe: Hotmart tiene webhook de notificación de venta + API REST para listar ventas. Ambos vías serían viables.

**Diseño candidato**:
```
Hotmart → POST /api/webhook/hotmart/<tenant_slug>
  → validar HMAC (Hotmart usa HOTMART-HOTTOK header)
  → match buyer email → findOrCreateCrmContact(client_id, email)
  → INSERT crm_sales con channel='hotmart'
  → eventBus 'sale.created'
```

**Riesgo**: Hotmart cambia API frecuentemente y la doc en español/portugués es regular. Asumir 2 semanas de pelea inicial.

**Quién**: pendiente asignar. Tracker en [[Sprint-Arreglos]] como T6.x.

---

### 4. Stripe self-service billing del SaaS

**Hipótesis**: hoy el SaaS Apex no se cobra automáticamente — cada cliente factura por contrato manual. Si quieres escalar a 30+ tenants, esto se ahoga. Stripe Subscriptions + un portal `central.blackwolfsec.io/billing` resolverían el cobro automático.

**Estado**: idea, sin diseño formal. Tabla `clients.billing_*` mencionada en [[Modelo-de-Negocio]] como oportunidad.

**Cuestión clave**: ¿pricing flat por tier o pricing por features (clientes con CRM only vs CRM+Formación+Mentorías)? La segunda opción es más justa pero mucho más compleja en Stripe (price IDs por feature combo).

**Spike sugerido**: empezar con flat por tier. 3 planes: Starter / Pro / Enterprise. Migrar a per-feature solo si los datos de uso justifican que algunos tenants solo usan 1-2 módulos.

---

### 5. Donna v2 — multi-channel y memoria persistente

**Hipótesis**: Donna hoy es "asistente Telegram". V2 sería "asistente cross-channel" (Slack, Telegram, web) con memoria de proyectos a largo plazo (no solo de contexto inmediato).

**Estado**: research. Memoria de Anthropic ya es persistente vía la propia capa pero hay un "cómo lo expones bien" que no está resuelto.

**Open questions**:
- ¿Memoria global compartida entre canales o silo por canal?
- ¿Cómo se reconcilian decisiones tomadas en Telegram con el state que vive en `brain_decisions`?
- ¿Cuánto pesa el token cost mensual si pasamos a context-grande con cada interacción?

**Tracking**: ver [[Donna-y-Army]] → "Próximas iteraciones".

---

### 6. AI Agents — Setter / Outreach / Email / Brain (afinado)

**Hipótesis**: los 4 agentes AI ya en producción ([[Modulo-AI-Agents]]) son MVP. Hay margen grande de mejora en:

- **Setter AI**: hoy responde con templates dinámicos. Probar full-LLM-driven con guardrails para conversiones más naturales.
- **Outreach AI**: la calidad del lead-scoring es regular. Probar embeddings + similarity ranking vs. los datos históricos del tenant.
- **Email AI**: redacta bien pero no aprende del estilo del director. Probar fine-tune ligero o RAG sobre emails históricos del tenant.
- **Brain**: el reasoning está siloado por commander. Investigar si vale "Brain global" que ve TODO el tenant antes de decidir.

**Estado**: cada uno en estado distinto. Setter es el más maduro, Brain el más experimental.

---

## Tech radar

| Tech | Status | Notas |
|---|---|---|
| Anthropic Claude (Sonnet/Haiku) | ✅ adopt | Ya core de Donna y Army. Seguir |
| Supabase | ✅ adopt | Core. RLS pendiente endurecer |
| Vercel | ✅ adopt | Deploy Dashboard-Ops |
| Cloudflare Tunnel | ✅ adopt | Enjambre-API expuesto. Robusto |
| Whatsapp-web.js | 🟡 hold | Funciona pero frágil. Ver área #2 |
| ManyChat | 🔴 drop | Marcado zombie. Ya no se usa. Quitar referencias del código |
| Stripe Subscriptions | 🟡 trial | Para self-service billing del SaaS (área #4) |
| Hotmart API | 🟡 trial | Para conector área #3 |
| pgvector | 🟢 assess | Para embeddings de Outreach AI / Email AI |
| Temporal.io | 🟢 assess | Para workflows largos del Brain (alternativa a sequence-worker) |
| WhatsApp Business Cloud API | 🟢 assess | Migración área #2 |
| LiteLLM proxy | 🟢 assess | Si crecemos en cost de Anthropic, abstraer detrás de proxy con caching |

`adopt` = en prod, seguir invirtiendo · `trial` = piloto activo · `assess` = evaluar pronto · `hold` = no más inversión, mantener · `drop` = quitar

---

## Cómo proponer una nueva línea de I+D

1. Crear archivo en `apex/I+D/<Nombre-Corto>.md` con frontmatter `tags: [apex, i+d, spike-or-research]`.
2. Estructura mínima: **Hipótesis** (1 párrafo) · **Estado** · **Spike sugerido** · **Open questions**.
3. Si la I+D madura y se decide construir, mover a `_moc/MOC-Roadmap` o `apex/Plataforma/<Modulo>` según corresponda.
4. Si se descarta, dejar el archivo con frontmatter `status: dropped` y razón. No borrar — el "por qué no" es valioso.

## Conexiones

- Plataforma core: [[Arquitectura-General]] · [[Donna-y-Army]] · [[Portal-Infoproducto]]
- Trabajo decidido (no I+D): [[_moc/MOC-Roadmap]] · [[Sprint-Arreglos]]
- Decisiones tomadas que afectan I+D: [[_moc/MOC-Decisiones-CTO]]
- Negocio que puede priorizar I+D: [[Modelo-de-Negocio]]
- I+D del producto hermano: [[I+D-Cargonex]]
