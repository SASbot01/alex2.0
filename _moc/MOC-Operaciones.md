---
tags: [moc, operaciones]
created: 2026-05-07
---

# MOC — Operaciones

Map of Content de la capa cognitiva (Donna + Army) y los flujos operativos automáticos.

## Capa cognitiva

- [[Donna-Telegram]] — asistente ejecutiva única, viva en `/opt/ai-agent-console`, habla por Telegram
- [[Army-Commanders]] — General + Comandantes por dominio (CRM activo, OPS/DEV/FORMS solo doctrina)
- [[Doctrines]] — los markdown que definen comportamiento
- [[Brain-Decisions]] — tabla `brain_decisions` donde cae todo el reasoning + AI usage

## Cost & observability

- [[AI-Cost-Tracking]] — tracking via `decision_type='ai_usage'` + endpoint `/api/ai-usage`
- [[Prompt-Caching]] — usado en orchestrator + commanders (cache_control ephemeral)

## Flujos automáticos

- [[Webhooks]] — Stripe, ManyChat (zombie), forms, central, soc
- [[Cron-Jobs]] — sequence-worker, brain-worker (extracción semanal)
- [[Cleanup-Mantenimiento]] — propuesta de mantenimiento periódico

## Conexiones

- Backend: [[Enjambre-API]]
- Decisiones que toma Donna: [[Brain-Decisions]]
- Cómo escalar a OPS/DEV/FORMS: [[Sprint-7-DX-Cleanup]] o [[Decision-Army-Completion]]
