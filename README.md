# Alex 2.0 — Cerebro de Alejandro / BlackWolf

Segundo cerebro en Obsidian para el CTO de BlackWolfSec. Centraliza todo el conocimiento sobre la empresa, la plataforma Dashboard-Ops, los clientes, la arquitectura técnica, las integraciones, las auditorías, el roadmap y las decisiones estratégicas.

## Cómo usar este vault

Abre [[HOME]] como punto de partida. Desde ahí navegas vía:

- **MOCs** (Maps of Content) en `_moc/` — mapas de cada gran área temática.
- **Carpetas numeradas** — cada una es una vertical: empresa, plataforma, clientes, tecnología, etc.
- **Wikilinks `[[Archivo]]`** — todo está cross-linkeado. Sigue los hilos.
- **Tags** `#critical`, `#deuda`, `#zombie`, `#cliente`, `#decision-cto` — filtros transversales.

## Estructura

```
00-Empresa/         → Quién somos, qué vendemos, cómo funcionamos
01-Plataforma/      → Dashboard-Ops + enjambre-api + módulos
02-Clientes/        → Brain de cada tenant en producción
03-Tecnologia/      → Stack, convenciones, deployment
04-Integraciones/   → Stripe, Resend, WhatsApp, Google, Meta, etc.
05-Operaciones/     → Donna, Army, doctrines, AI cost tracking
06-Auditoria-2026-05-07/  → Estudio integral de la plataforma
07-Roadmap/         → Sprints 0-8 hacia "SaaS empresarial vendible"
08-Decisiones-CTO/  → ADRs y políticas
09-Playbooks/       → Procedimientos operativos repetibles
10-Glosario/        → Términos, tipos de cliente, acrónimos
99-Personal/        → Sobre Alejandro, decisiones, diario
```

## Convenciones del vault

- **Frontmatter mínimo**: `tags`, `created`, `updated`, `status`.
- **Wikilinks** para todo lo interno: `[[Hugo-EnFormaConHugo]]`.
- **Markdown para código**, español de España para prosa.
- **Fecha absoluta** siempre (2026-05-07), nunca "hace dos semanas".
- **Honestidad**: si algo está roto, se documenta como roto.

## Última actualización

- **2026-05-07** — Bootstrap del cerebro tras auditoría integral post-Hito 5 del Portal Infoproducto.
