---
tags: [plataforma, donna, army, ai]
created: 2026-05-07
---

# Donna + Army (Capa Cognitiva)

Sistema de agentes IA para automatizar operaciones. **Donna** es la asistente ejecutiva única que habla por Telegram. **Army** es su backend de comandantes especializados por dominio.

## Arquitectura

```
Alejandro → Telegram → Donna (asistente) → HTTP → Dona General (planner)
                                                         ↓
                                       ┌─────────────────┼─────────────────┐
                                       ▼                 ▼                 ▼
                                  CRM Cmdr          OPS Cmdr         FORMS Cmdr
                                  (operativo)    (doctrina)        (doctrina)
                                       ↓
                                  Tools (CRM)
                                  brain_decisions
```

## Donna

- **Vive en**: `/opt/ai-agent-console` (memoria del proyecto)
- **Habla por**: Telegram con Alejandro
- **Coordina**: Enjambre + Wolf Trader + SOC + Dashboard-Ops
- **Identidad**: única entidad cognitiva. Aliases legacy "dona-general" no son entidad separada.

## Army (en `enjambre-api/src/army/`)

### General — `general.js`

- Daemon Fastify en puerto 8789
- Endpoints: `POST /order`, `GET /status`, `GET /decisions`, `GET /health`
- Auth: `X-Peer-Token` validado contra `DONA_PEER_TOKEN` env var
- Modelo: `claude-sonnet-4-5-20250929` (configurable via `AGENT_GENERAL_MODEL`)
- Recibe orden → planifica con Claude → asigna a Comandantes vía `eventBus`

### Comandantes

`enjambre-api/src/army/commanders/`:

| Commander | Estado | Doctrina | Modelo |
|---|---|---|---|
| `crm.js` | ✅ Operativo | `doctrine/crm.md` | haiku-4-5 (configurable) |
| `ops.js` | 🔴 Solo doctrina | `doctrine/ops.md` | (no implementado) |
| `dev.js` | 🔴 Solo doctrina | `doctrine/dev.md` | (no implementado) |
| `forms.js` | 🔴 Solo doctrina | `doctrine/forms.md` | (no implementado) |

**Si Dona delega a OPS/DEV/FORMS, falla silenciosamente.** Ver [[Flaquezas-Estructurales]] punto 8.

### BaseCommander

`enjambre-api/src/army/base-commander.js` (~300 LOC):
- Agentic loop con tool calling
- **Prompt caching** (`cache_control: { type: 'ephemeral' }`) en system + tools
- Heartbeat en DB
- Métodos: `_tick()`, `_doMission()`, `_report()`
- Modo dry-run (`ARMY_DRY_RUN=1`)

### Doctrines (`src/army/doctrine/*.md`)

- `general.md` — estrategia de descomposición y asignación de Comandantes
- `crm.md` — lead pipeline, setter assignment, tagging
- `ops.md` — revenue, anomalías, presupuesto
- `dev.md` — incidentes, despliegues, performance
- `forms.md` — intake, validación, routing

Los markdowns son contenido del prompt caché del BaseCommander. Cambios → invalidar caché → recompilar prompt.

## Brain decisions

Tabla `brain_decisions` registra todo el reasoning:

```sql
CREATE TABLE brain_decisions (
  id uuid,
  decision_type text,        -- 'ai_usage', 'crm_decision', 'ops_decision', etc.
  agents_involved text[],
  reasoning text,
  actions_taken jsonb[],
  confidence numeric,        -- (en 'ai_usage' se usa para guardar cost_usd)
  created_at timestamptz
);
```

`decision_type='ai_usage'` registra cada llamada a Claude con `agent`, `model`, tokens, `cost_usd`. Endpoint `/api/ai-usage` agrega por agent/model/día.

## SOUL.md

`enjambre-api/src/army/SOUL.md` documenta la lógica conceptual del sistema. Vale la pena releer cada vez que se vaya a tocar Army.

## Donna→Dona bridge — INEXISTENTE

Memoria proyecto + auditoría confirman:
- Donna NO tiene tool para invocar `POST /order` de Dona General
- El "executor" del lado de Donna no está construido
- Hoy el bridge existe en teoría (SOUL.md), no en código

Esto significa que **Army CRM corre autónomo en patrullas (cada 300s)**, pero las órdenes ad-hoc desde Telegram → Dona General no llegan.

## Tabla `general_orders` referenciada y no creada

`general.js` referencia `general_orders` (audit de órdenes) pero la migration nunca se creó. INSERTs fallan silenciosamente.

## Pendientes para que Army esté completo

1. **Construir tool en Donna** para invocar `POST /order` de Dona General
2. **Implementar handlers** OPS, DEV, FORMS o quitar las doctrinas
3. **Crear migration `general_orders`** con audit trail
4. **Aprobación humana** antes de mutaciones irreversibles (hoy puede ejecutar tools sin confirmación)

## Conexiones

- Backend: [[Enjambre-API]]
- Brain decisions tabla: [[Brain-Decisions]]
- Cost tracking: [[AI-Cost-Tracking]]
- Operación: [[_moc/MOC-Operaciones]]
