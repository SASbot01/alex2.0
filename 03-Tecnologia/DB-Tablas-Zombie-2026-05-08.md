---
tags: [tecnologia, db, deuda, zombie]
created: 2026-05-08
updated: 2026-05-08
audit_method: pg_stat_user_tables
---

# DB Tablas Zombie / Dormant — Audit 2026-05-08

Sprint Arreglos `[db] Documentar 16 tablas DB no usadas`. Audit real: hay **48 tablas con 0 rows + 37 con 1-4 rows = 85 tablas zombie/dormant** (no las 16 estimadas).

## Categorización

### A. Sequences — fragmentación crítica (5 zombies)

Hay **3 sistemas paralelos de sequences** en la BD:
- `sequences` (0) + `sequence_enrollments` (0)
- `email_sequences` (1) + `email_sequence_contacts` (0)
- `crm_sequences` (0) + `crm_sequence_enrollments` (0)

**Acción**: consolidar a UN sistema (probablemente `crm_sequences` si es el más reciente). Drop los otros 4.

### B. Cerebro/Agent legacy — superseded por brain_decisions (12 tablas)

`brain_decisions` (2056 rows, vivo) reemplazó a:
- `cerebro_events` (0), `cerebro_actions` (0), `cerebro_learnings` (0)
- `cerebro_usage` (2), `cerebro_conversations` (2), `cerebro_messages` (4)
- `autonomous_agent_runs` (0), `agent_status` (0)
- `agent_performance_metrics` (0), `agent_evolution_snapshots` (0)
- `agent_decisions` (1), `agent_feedback` (1), `agent_learnings` (1)

**Acción**: confirmar con dev si `brain_decisions` cubre todo. Si sí → drop las 12. ~1MB liberado.

### C. Asesoria Suiza — webinar dormant (4 tablas, 0 rows)

Creadas para webinar 1-may-2026 (ya pasó):
- `asesoriasuiza_webinar_call_intake`
- `asesoriasuiza_establecimiento_submissions`
- `asesoriasuiza_webinar_signups`
- `asesoriasuiza_job_offers`

**Acción**: mantener (pueden activarse en próximas campañas). Si en 90d siguen 0 → drop.

### D. Portal Infoproducto — esperando merge PR #22 (4-5 tablas)

`portal_otp_codes`, `portal_users`, `marketplace_applications`, `marketplace_jobs`, `training_progress` — todo 0 rows porque el endpoint no está deployed aún (PR #22 sin mergear).

**Acción**: mantener. Tras merge PR #22 + tráfico real, re-auditar en 30d.

### E. BlackWolf internal — pre-uso (5 tablas)

- `bw_client_deliverables`, `bw_support_tickets`, `bw_ticket_messages`, `bw_contracts` (todo 0)
- `bw_client_projects` (4)

**Acción**: features pre-uso. Mantener. Si en 60d siguen <5 rows → revisar si la feature se va a ejecutar.

### F. Chat module — fantasma (5 tablas)

- `chat_messages` (1), `chat_conversations` (1), `chat_flows`, `chat_contacts`, `chat_broadcasts` (0)

Solo 2 rows totales. Parece feature scrapped.

**Acción**: investigar git log → probable drop.

### G. ERP module — building (5 tablas)

- `erp_stock_moves` (0), `erp_production_orders` (1), `erp_companies` (2), `erp_employees` (4), `erp_invoices` (3)
- `erp_contacts` (1002 rows, NO zombie)

**Acción**: módulo en construcción. Mantener.

### H. Auth/onboarding zombies (varios)

- `auth_sessions` (0) — feature no usada (frontend usa localStorage).
- `calendly_auth` (0) — antes de Google Calendar nativo.
- `signups` (0)
- `payment_links` (0)
- `invoices` (0)

**Acción**: investigar caso por caso. `calendly_auth` probable drop (ya migrado a GCal).

### I. CEO/founder dashboard zombies

- `ceo_integrations` (0), `ceo_finance_entries` (1)

**Acción**: feature pre-uso o scrapped. Investigar.

### J. Workflow zombies

- `workflow_webhook_triggers` (0)
- `linkedin_daily_reports` (0), `linkedin_posts` (1)
- `mentorias` (0), `copies_guiones` (0)

**Acción**: investigar git log para fecha de creación + último uso.

### K. Misc zombies (mantener si <30d desde creación, drop si >90d)

- `fireflies_transcripts` (0) — esperando integration Fathom.
- `legal_documents` (0)
- `leads` (0) — duplicado con `crm_contacts`?
- `setters` (0)
- `task_messages` (0)
- `agent_status` (0)
- `comunidad_messages` (3)
- `formacion_cursos` (3) vs `training_routes`/`training_formations` (1 cada)

## Tabla resumen

| Categoría | Tablas | Acción recomendada |
|---|---:|---|
| Sequences fragmentadas | 5 | Consolidar a 1, drop 4 |
| Cerebro legacy | 12 | Confirmar superseded → drop |
| Asesoria webinar | 4 | Mantener 90d |
| Portal pre-deploy | 5 | Esperar PR #22 + 30d |
| BlackWolf internal | 5 | Mantener 60d |
| Chat fantasma | 5 | Investigar → probable drop |
| ERP building | 5 | Mantener |
| Auth/onboarding zombies | ~5 | Caso por caso |
| CEO/founder | 2 | Investigar |
| Workflow zombies | ~5 | Investigar |
| Misc | ~10 | Caso por caso |

**Total estimado para drop**: ~25-30 tablas tras investigación. Liberaría ~5MB y reduce ruido en migraciones futuras.

## Procedimiento drop seguro

Para cada tabla candidata a drop:

1. **Verificar references**: `git grep <tabla> src/ api/ supabase/` — si aparece, NO drop.
2. **Verificar FKs**: `SELECT … FROM information_schema.table_constraints WHERE constraint_type = 'FOREIGN KEY' AND ... ` para ver tablas que la apuntan.
3. **Backup snapshot**: dump del schema y data (aunque sea 0 rows, conservar shape para rollback).
4. **Drop con migration**: `DROP TABLE IF EXISTS <tabla>` en migration numbered + `ROLLBACK: CREATE TABLE …`.
5. **Verificar logs 24h post-drop** — Sentry capture si algún code path la consultaba sigilosamente.

## Cómo replicar este audit

```bash
SUPABASE_MGMT=$(grep -E '^SUPABASE_MANAGEMENT_TOKEN=' /home/blackwolfsec/ejambre/.env | cut -d= -f2)
PROJECT_REF="kyxupfowsfkyqklxhtyo"
SQL="SELECT relname, n_live_tup, n_tup_ins, pg_size_pretty(pg_total_relation_size(relid)) AS size
     FROM pg_stat_user_tables WHERE schemaname='public' ORDER BY n_live_tup ASC;"
curl -X POST -H "Authorization: Bearer $SUPABASE_MGMT" -H "Content-Type: application/json" \
  "https://api.supabase.com/v1/projects/$PROJECT_REF/database/query" \
  -d "{\"query\": \"$SQL\"}"
```

## Conexiones

- Audit fuente: `pg_stat_user_tables` query `n_live_tup` ASC.
- Sprint Arreglos: `[db] Documentar 16 tablas DB no usadas` (2h, low).
- Tarea futura recomendada: `[db] Drop sequences fragmentadas (consolidar)` — estimación 4h con investigación + drop seguro.
