---
tags: [cargonex, i+d, investigacion, roadmap-tecnico]
created: 2026-05-08
updated: 2026-05-08
---

# I+D — Cargonex

Investigación y desarrollo abiertos sobre la línea [[Cargonex]]. Aquí viven las apuestas técnicas que aún no son producción del SaaS de logística: spikes, prototipos, hipótesis a validar, tech radar.

> Cargonex es un producto **joven** (1 cliente live en 2026-05). La mayor parte del backlog es de "endurecer lo que hay" más que de explorar lo nuevo. Aún así, hay vectores de I+D que merecen pensarse antes de que el cliente número 2 cambie el cost de equivocarse.

## Áreas activas

### 1. Migración Eilers de `db.json` → `tenant-<id>.json`

**Hipótesis**: el flag `legacy:true` de [[Eilers-Logistik]] resuelve el día-1 (los 8343 records se quedan en `db.json`) pero crea heterogeneidad operativa permanente. Cada vez que se haga un script cross-tenant ("export todos los KPIs", "calcular billing por tenant"), hay que ramificar `if legacy then path A else path B`. Migrar es dolor único; mantener es dolor distribuido.

**Estado**: research. Diseño no escrito.

**Spike sugerido**:
```
1. Crear tenant-cli_eilers0001.json copiando shape de db.json
2. Cron diario que sincroniza db.json → tenant-cli_eilers0001.json (one-way, db.json sigue siendo source of truth)
3. Cuando convergen estables 2 semanas, flip:
   - tenant.legacy = false
   - getTenantDataFile(eilers) ahora retorna tenant-cli_eilers0001.json
   - db.json se queda como backup read-only
4. Eliminar la rama legacy:true del código un sprint después
```

**Riesgo**: 8343 records. Un bug de copia silencioso = pérdida de datos del cliente fundador. Necesita validador estricto antes y después de cada copia.

**Trigger para hacerlo**: cuando aterrice el cliente Cargonex número 2 — en ese momento la diversidad legacy/non-legacy duele en serio.

---

### 2. Tour Planner — afinado del prompt y memoria de rutas

**Hipótesis**: el tour planner usa Anthropic `claude-haiku-4-5` con un prompt baseline. Hay margen para:
- Few-shot con rutas históricas exitosas del tenant (RAG sobre `tenant-<id>.json#records`).
- Considerar restricciones específicas del cliente (tiempos de descanso DE EN12, ventanas de carga, OBD2 telemetry de los vehículos).
- Output estructurado JSON validado para reducir parse errors.

**Estado**: en producción con prompt v1. KPIs de calidad no medidos sistemáticamente — hay que instrumentar antes de optimizar.

**Spike sugerido**: añadir endpoint `POST /api/tour-planner/feedback` para que el dispatcher marque `accepted/rejected/edited` en cada plan. Acumular 200+ samples → fine-tune o prompt v2 basado en patrones.

**Open question**: ¿el cliente paga por tokens de Anthropic o BlackWolf absorbe el coste? Si lo segundo, hay un cap implícito en cuánto se puede experimentar con prompts caros. Hoy `claude-haiku-4-5` mantiene cost bajo, pero si v2 quiere `sonnet`, hay que renegociar.

---

### 3. OBD2 — telemetría más profunda

**Hipótesis**: hoy se almacenan datos básicos de OBD2 (kilometraje, faults). El device puede dar mucho más: presión neumáticos, consumo instantáneo, temperatura motor, behavior driving (aceleración/frenado bruscos). Eso permitiría:
- Mantenimiento predictivo (alertar antes de fallo)
- Coaching de drivers (por estilo de conducción)
- Cálculo real de coste por km vs. flat

**Estado**: device hardware ya conectado (memoria proyecto: "OBD2 wired" desde fase C). Solo se está consumiendo el subset mínimo. Investigar qué más expone el device sin upgrades de hardware.

**Spike sugerido**: dump 1 semana de raw OBD2 stream de un vehículo de prueba. Análisis offline para ver qué señales son ruido y cuáles son útiles. Después diseñar la tabla `obd_telemetry` con las útiles + retención apropiada (probablemente 30 días raw + agregados mensuales perpetuos).

**Riesgo DSGVO**: behavior driving es dato personal del driver. Necesita consentimiento explícito en contrato laboral. Coordinar con [[Eilers-Logistik#DSGVO / GDPR]].

---

### 4. DSGVO — automatización de export/delete por sujeto

**Hipótesis**: GDPR Art. 15 (right of access) y Art. 17 (right to be forgotten) son obligación legal con plazo 30 días. Hoy si Eilers recibe un request de un driver, el procedimiento es manual (grep + redact + send). Eso no escala y no es auditable.

**Estado**: pendiente. Es bloqueante para go-live formal con Eilers (ver [[Eilers-Logistik#DSGVO / GDPR]]).

**Spike sugerido**: endpoint `POST /api/platform-admin {action:"gdpr-export", tenantId, subjectEmail}` que:
- Lista cada fila en `tenant-<id>.json` que contenga el email del sujeto
- Empaqueta en zip con README.md humano-leíble
- Audit entry con `action:"gdpr-export"`
- Devuelve URL temporal de descarga

Y simétrico `gdpr-delete` con dry-run + confirmación.

**Open**: ¿qué hacemos con datos cross-referenced (un driver mencionado en un record de otro driver)? Anonymize en vez de delete. Decidir política antes de codear.

---

### 5. Mobile UX — PIN, instalación PWA, offline

**Hipótesis**: Cargonex es mobile-first. Los drivers usan el portal en su móvil de trabajo. Hoy:
- El PIN flow no se ha probado en device real (solo smoke local en :3099). Riesgo de fallo silencioso.
- La instalación PWA ("Add to Home Screen") existe pero no se enseña al usuario activamente.
- El modo offline funciona vía SW pero no hay UX clara de "estás offline, los cambios se sincronizan cuando vuelvas".

**Estado**: pendiente test en device real. Probablemente revele 2-3 bugs.

**Spike sugerido**: día completo con un dispositivo Android estándar de driver (no el iPhone del CTO) probando el flujo PIN end-to-end. Documentar cada fricción. Iterar.

**Quién**: pendiente. Idealmente alguien que NO sea desarrollador para que vea la fricción real.

---

### 6. Multi-language — expansión EU

**Hipótesis**: Eilers es DE primary, EN/ES disponibles. Si Cargonex aspira a otros mercados europeos (Francia, Italia, Holanda), añadir FR/IT/NL ahora cuesta poco; añadirlo cuando haya 5 tenants productivos con strings hardcoded cuesta mucho.

**Estado**: i18n cubierto en stations DNM4/DNM6/DNW9/HNM6/DHL/FEDEX/JDHH para DE/EN/ES (commit 380533b del repo). Falta auditar el resto del SPA + landing.

**Spike sugerido**: pasar `i18n-engine.js` por extractor automático que liste todos los strings literales en JSX/HTML que NO pasen por `L()` o equivalente. Lista debe ser cerrable en 1-2 sprints.

**Open**: ¿quién traduce? ¿Profesional, DeepL + revisión, o nativo del país cuando aterrice un tenant local? Lo tercero es lo más escalable pero retrasa go-live por mercado.

---

### 7. Stripe Connect para self-service billing

**Hipótesis**: hoy Eilers paga por contrato anual offline. Si Cargonex quiere ser un producto vendible sin overhead operativo, necesita billing integrado tipo Vercel/Linear (registro web → tarjeta → tenant aprovisionado).

**Estado**: idea. Sin diseño.

**Diseño candidato**:
```
1. Marketing landing añade "Try free for 14 days" CTA
2. Registro web → Stripe Customer creado en cuenta BlackWolf
3. Tenant aprovisionado en pending hasta confirmar tarjeta
4. Trial 14 días, después cobra mensual (3 tiers: starter / pro / enterprise)
5. Webhook stripe.subscription.updated → flip tenant.status (active / paused / revoked)
```

**Trigger**: cuando llegue el tenant Cargonex número 2 cliente con presupuesto bajo / proceso de compra ágil. Eilers es enterprise-ish, otros pueden ser SMB.

**Open**: ¿Stripe Connect (cliente paga directo a su Stripe account) o Stripe Billing único (BlackWolf cobra y reparte)? Lo segundo es más simple para Cargonex porque la oferta es del producto, no del cliente. Diferencia con [[Stripe]] de Hugo (que SÍ es Connect porque Hugo cobra a SUS alumnos).

---

### 8. Tenant-aware push notifications

**Hipótesis**: hoy `push-subs.json` es un único bag. Si un mensaje se broadcast, lo reciben drivers de todos los tenants (en escenarios multi-tenant futuros, pánico). Hay que aislar por `clientId`.

**Estado**: trabajo de hardening de baja prioridad — solo aplica cuando entre el tenant número 2.

**Spike sugerido**: cambiar shape de `push-subs.json` a `{ <clientId>: [<sub>, ...] }`. Migrar las subs existentes asignándolas a Eilers. Endpoint `/api/push-send` ya recibe `X-Client-ID`, solo hay que filtrar antes de enviar.

---

## Tech radar

| Tech | Status | Notas |
|---|---|---|
| Express + Postgres + JSON | ✅ adopt | Stack base. Mantener simple |
| Vanilla JS PWA + SW | ✅ adopt | Funciona, no introducir framework |
| Cloudflare Tunnel | ✅ adopt | Robusto, sin puertos públicos |
| Anthropic claude-haiku-4-5 | ✅ adopt | Tour planner. Coste bajo |
| Resend SMTP | 🟡 trial | Cutover en progreso (verificación dominio) |
| Stripe Connect | 🟢 assess | Self-service billing — área #7 |
| pgvector | 🟢 assess | Embedding de rutas históricas para tour planner — área #2 |
| WebPush per-tenant | 🟢 assess | Aislamiento subs — área #8 |
| Anthropic Sonnet | 🟢 assess | Si haiku se queda corto en tour planner v2 |
| Workers / Background jobs framework | 🟢 assess | Hoy todo síncrono. Cuando llegue el cliente N2 con cron-jobs (alerts, reportes), evaluar (BullMQ vs cron+lock vs Temporal) |
| React / Vue | 🔴 drop | Tentación de "modernizar" el SPA. NO. Vanilla funciona perfecto, framework solo añade peso de bundle |

`adopt` = en prod, seguir invirtiendo · `trial` = piloto activo · `assess` = evaluar pronto · `hold` = no más inversión, mantener · `drop` = quitar / no introducir

---

## Cómo proponer una nueva línea de I+D

1. Crear archivo en `cargonex/I+D/<Nombre-Corto>.md` con frontmatter `tags: [cargonex, i+d, spike-or-research]`.
2. Estructura mínima: **Hipótesis** (1 párrafo) · **Estado** · **Spike sugerido** · **Open questions**.
3. Si la I+D madura y se decide construir, mover a `cargonex/Plataforma/` o referenciar desde [[_moc/MOC-Roadmap]].
4. Si se descarta, dejar el archivo con frontmatter `status: dropped` y razón. No borrar.

## Conexiones

- Producto: [[Cargonex]] · [[Arquitectura]]
- Cliente fundador (donde se valida casi toda la I+D): [[Eilers-Logistik]]
- Decisiones que enmarcan la I+D: [[ADR-Cargonex-Stack-Separado]]
- Onboarding nuevo tenant: [[Playbook-Crear-Tenant-Cargonex]]
- I+D del producto hermano: [[I+D-Apex]]
- Negocio que puede priorizar I+D: [[Modelo-de-Negocio]]
