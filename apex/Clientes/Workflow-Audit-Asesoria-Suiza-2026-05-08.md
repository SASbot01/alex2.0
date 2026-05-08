---
tags: [workflows, audit, asesoria-suiza, sprint-planning]
slug: asesorias-suiza
created: 2026-05-08
updated: 2026-05-08
type: audit
sprint_status: sprint-1-done
---

# Audit de Workflows — Asesoría Suiza (2026-05-08)

Catálogo completo de event-points donde el funnel pierde métricas o procesos. Generado tras detectar que **49 bookings se quedaron sin sync al CRM durante 30+ días** porque la configuración de `target_pipeline_slug` en `booking_hosts` estaba vacía y el fallback apuntaba a un pipeline inexistente.

## Contexto disparador

User Alejandro pidió "analizar todos los posibles workflows que se pueden crear para no perder métricas ni procesos", arrancando por:
- Bookings con hosts Adrián / Toñi / Mio (Portillo) / Lukas → meter al lead en stage `llamada_agendada` del pipeline correspondiente.
- Bookings de seguros (web Asesoría Suiza) → pipeline correspondiente.
- Pixel pendiente en las webs de seguros.

El audit fue más allá del request literal y mapeó **6 categorías × 30 workflows** posibles. Sprint 1 cerró el agujero crítico (booking sync). Sprints 2-4 cubren el resto en orden de impacto.

---

## Sprint 1 ✅ DONE — Booking sync (recovery)

Ver [[AsesoriaSuiza-Portillo-Lukas#sprint-2026-05-08-booking-sync-recuperado--nueva-pipeline-cierre]] para el detalle.

**Resultado:** 49/49 bookings linkeados, 6 hosts configurados, pipeline closers nueva, endpoint sin bug del fallback.

---

## Catálogo completo (30 workflows)

### A. Booking lifecycle

| # | Evento | Acción CRM | Sprint |
|---|---|---|---|
| A1 | `booking confirmado` | mover/crear contacto a *pipeline.host* / `llamada_agendada` + setear `fecha_llamada`, `assigned_closer`, link booking | ✅ Sprint 1 |
| A2 | `booking cancelado` (pre-llamada) | activity + stage `cancelado_pre_llamada` (o devolver a `nuevo_lead`) + WA "¿reagendamos?" | Sprint 2 |
| A3 | `booking reschedule` | activity "reagendado de X a Y" + actualizar `fecha_llamada` | Sprint 2 |
| A4 | `booking no-show` (cron 1h post `start_at`, sin `attended_at`) | stage `ghosting` + activity + WA recover | Sprint 2 |
| A5 | `booking attended` (manual o via Fathom) | stage `en_conversacion` o `seguimiento_cierre` + activity | Sprint 3 |
| A6 | `reminder enviado / abierto` | activity timeline | Sprint 3 |
| A7 | `host-routing automático para insurance webs` | si lead llega de `/seguros/*` → forzar pipeline seguros, no Europeos | Sprint 2 |

### B. Form lifecycle (entrada de leads)

| # | Evento | Acción | Sprint |
|---|---|---|---|
| B1 | Form Trabajo Portillo/Lukas submit | crm_contact en `FormTrabajoX/lead` con custom_fields `trabajo_*` | ✅ ya en prod |
| B2 | Form Webinar agendar (8 preguntas) | crm_contact en `FormWebinarX/inscrito` + activity | Sprint 3 — verificar si funciona |
| B3 | Form `con-trabajo` (upload Anmeldung) | crm_contact en `EstablecimientoX/anmeldung` + asset upload tracking | Sprint 3 |
| B4 | Lead magnet `/suizatrabajo` (49 empresas) | log "vio_lead_magnet" + activity (hoy se pierde) | Sprint 3 |
| B5 | Click "Enviar por Gmail" en lead magnet | event "applied_to_company:<empresa>" para correlacionar respuestas | Sprint 4 |

### C. Pago lifecycle

| # | Evento | Acción | Sprint |
|---|---|---|---|
| C1 | `checkout.session.completed` | upsert sale + auto-mover trabajo→clientes_portillo/inscrito | Sprint 2 (parcial hoy) |
| C2 | `invoice.payment_failed` / `charge.failed` | activity + alerta WA al operator + retry email | Sprint 2 |
| C3 | `charge.refunded` | mover a stage `reembolsado` + activity | Sprint 2 |
| C4 | `payment_link clicked` (UTM tracking) | activity "intent_pago" para detectar abandonos | Sprint 4 |
| C5 | Sin pagos en X días tras `inscrito` | mover a `abandonado` + email recover | Sprint 4 |

### D. Tracking server-side (CAPI híbrido)

| # | Evento | Trigger | Destino |
|---|---|---|---|
| D1 | `Lead` | form Trabajo / Webinar / Seguros | Meta CAPI + GA4 + TikTok |
| D2 | `Schedule` | A1 booking confirm | Meta + GA4 + TikTok (value=0) |
| D3 | `Purchase` | C1 Stripe paid | Meta + GA4 + TikTok (value=CHF) |
| D4 | `LeadMagnet` | B4 listado empresas visto | Custom event |
| D5 | `Anmeldung` | B3 upload | Custom event (señal calidad alta) |

**Patrón implementación:** `lib/tracking/capi.js` + tabla `tracking_events` (`id, contact_id, event_name, event_id, payload, sent_at_meta, sent_at_ga, sent_at_tiktok`). Cada workflow `INSERT` y un cron empuja con dedupe vía `event_id` = `${contact_id}-${event_name}-${timestamp}`. Sprint 2.

### E. Cross-operator routing

| # | Evento | Acción | Sprint |
|---|---|---|---|
| E1 | Lead llega a host sin operador | round-robin entre Adrián/Toñi/Jose | Sprint 4 |
| E2 | Lead repetido (mismo email reagenda otro host) | mantener owner original, NO reasignar `assigned_closer` | Sprint 4 |
| E3 | Lead reserva 2 hosts en 7 días | merge + flag `dup_reserva=true` para WA recovery | Sprint 4 |

### F. Métricas y observability

| # | Evento | Acción | Sprint |
|---|---|---|---|
| F1 | Dashboard "leads → llamadas → asistidas → ventas" por `assigned_closer` | enriquecer `OperatorDetail.jsx` (ya existe sin booking metrics) | Sprint 3 |
| F2 | Alert WA si `bookings_no_sync_24h > 0` | cron health-check al webhook | Sprint 2 |
| F3 | Alert si tasa cancelación > 20% en 7d por host | telemetría a Donna | Sprint 4 |
| F4 | Reporte semanal Telegram | revenue + #llamadas + #ventas + top fuente | Sprint 4 (parcial vía Donna) |

### G. Lifecycle post-venta

| # | Evento | Acción | Sprint |
|---|---|---|---|
| G1 | Cliente nuevo en Stripe | hilo seguimiento + plantilla bienvenida WA | Sprint 4 |
| G2 | 30/60/90 días desde alta | check-in automático "¿cómo va con el seguro/proceso?" | Sprint 4 |
| G3 | Cliente sin actividad 90d | etiqueta `dormant` + secuencia upsell | Sprint 4 |
| G4 | Renovación seguro Lukas (`fecha_renovacion - 60d`) | activity + WA aviso | Sprint 4 (alto ROI Lukas) |

---

## Roadmap propuesto

**Sprint 2 (1-2 semanas):**
- A2/A3/A4 (cancel/reschedule/no-show via cron)
- A7 (forzar pipeline seguros desde webs específicas)
- C2/C3 (Stripe failure + refund automation)
- F2 (health-check booking sync)
- D1-D5 + tabla `tracking_events` (preparar la base para el pixel)

**Sprint 3 (1 semana):**
- B2/B3 (Webinar + Anmeldung lifecycle)
- A5/A6 (attended + reminder ack)
- F1 (dashboard métricas closer)
- B4 (lead magnet tracking)

**Sprint 4 (long-tail, mientras se ejecutan los anteriores):**
- E1/E2/E3 (routing closers)
- G1-G4 (post-venta, especial G4 Lukas para renovaciones)
- C4/C5 (intent y abandono pago)
- F3/F4 (alertas + reportes)

---

## Decisiones registradas

- **2026-05-08 — Pipeline closers compartida**: Adrián/Toñi/Jose comparten `Cierre Closers · Portillo` (no pipeline por closer). Atribución vía `assigned_closer` (slug). Reasoning: 3 closers que rotan según disponibilidad, fragmentar pipelines impide ver el funnel global de "cierres pendientes" y obliga a hacer `UNION` en cada query. Si en el futuro hay reporting individual fuerte, se filtra por `assigned_closer`.
- **2026-05-08 — Tracking híbrido**: pixel cliente (rápido, deploy en horas) + CAPI server-side (resistente a adblock, dedupe por `event_id`). El server-side se construye en Sprint 2 sobre la tabla `tracking_events` que se alimenta de cada workflow.
- **2026-05-08 — Lukas seguros usa key armonizada `llamada_agendada`**: la pipeline Lukas tenía la stage como `key='contacted', label='📞 Llamada Agendada'`. Renombrado a `llamada_agendada` para coherencia con el resto del tenant (0 contactos afectados).

---

## Conexiones

- Cliente: [[AsesoriaSuiza-Portillo-Lukas]]
- Plataforma: [[Dashboard-Ops]] · [[Booking-Engine]] (a crear)
- Patrón: [[Multi-Owner]] · [[CRM-Pipeline-Routing]] (a crear)
- Integración futura: [[Meta-CAPI]] · [[GA4]] · [[TikTok-Events-API]] (todas a crear)
