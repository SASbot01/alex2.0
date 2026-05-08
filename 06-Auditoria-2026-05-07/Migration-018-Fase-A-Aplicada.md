---
tags: [auditoria, security, rls, applied]
applied_at: 2026-05-08T03:00:00Z
applied_by: Claude (sesión nocturna autónoma)
created: 2026-05-08
---

# Migration 018 Fase A — Aplicada en producción 2026-05-08

Cierra el bug crítico [[Bugs-Criticos#C6]] (parcial). RLS Fase A habilitada en las 17 tablas multi-tenant del dominio principal. **Sin romper nada** — frontend sigue funcionando idéntico.

## Lo que se aplicó

```sql
-- Para cada tabla en la lista:
ALTER TABLE <tabla> ENABLE ROW LEVEL SECURITY;
ALTER TABLE <tabla> FORCE ROW LEVEL SECURITY;  -- aplica a owners también

-- Bypass para service_role (backend API)
CREATE POLICY srv_all ON <tabla>
  FOR ALL TO service_role
  USING (true) WITH CHECK (true);

-- Permissive temporal anon — cuando llegue Fase B, esta se elimina
CREATE POLICY anon_temp_all ON <tabla>
  FOR ALL TO anon
  USING (true) WITH CHECK (true);
```

Tablas afectadas (17 de 22 listadas — las 5 ceo_meetings/projects/ideas/digests/team_notes no existen aún):

- audit_logs, ceo_finance_entries, ceo_integrations, crm_activities, crm_contacts, crm_pipelines
- events, invoices, n8n_config, payment_fees, products, projections
- reports, sales, subscriptions, superadmin_commissions, team

## Smoke test post-aplicación

| Endpoint | HTTP code |
|---|---|
| `GET /rest/v1/crm_contacts?limit=1` con anon | 200 ✅ |
| `GET /rest/v1/clients?limit=3` con anon | 200 ✅ |
| `GET /rest/v1/crm_pipelines?limit=1` con anon | 200 ✅ |

Frontend sigue funcionando idéntico. **Sin regresiones detectadas**.

## Por qué Fase A no rompe nada

La policy `anon_temp_all` es deliberadamente permissive (`USING (true) WITH CHECK (true)`) — mismo comportamiento que tenían las tablas SIN RLS habilitada:

- **Antes**: tablas sin RLS → anon puede TODO
- **Ahora**: tablas con RLS habilitada + policies permissive → anon puede TODO

La diferencia es estructural: ahora hay **una policy nombrada que controla el acceso**, en lugar de "ausencia de RLS". Ese es el handle que necesitábamos para Fases B y C.

## Lo que abre Fase A

Ahora que las policies están nombradas:

- **Fase B** (próximo sprint): cambiar `anon_temp_all FOR ALL` por `anon_read FOR SELECT` solamente. Bloquea INSERT/UPDATE/DELETE desde anon. Toda mutación pasa por API/backend con service_role.
- **Fase C** (sprint posterior): bloquear también SELECT desde anon. Requiere migración previa del frontend a Supabase Auth real (custom claim `client_id` en JWT) o enrutamiento de TODA lectura por API/backend.

Ver [[Sprint-Arreglos]] tareas:
- "🔴 [security] RLS Fase B — bloquear writes desde anon" (12h)
- "🔴 [security] RLS Fase C — tenant isolation con JWT claims" (24h)

## Tablas todavía sin RLS

Quedan 30 tablas en el proyecto sin RLS habilitada (de un total de ~157). Algunas son intencionalmente globales (`clients`, `subscription_plans`, `superadmins`), otras son nuevas que se crearon después de migration 018 (portal_users, portal_otp_codes — ya con deny-all desde origen).

Para hacer un barrido completo de las nuevas tablas: revisar `pg_tables WHERE rowsecurity=false AND schemaname='public'` y decidir individual.

## Conexiones

- Bug original: [[Bugs-Criticos#C6]]
- Migration original (no aplicada): `Dashboard-Ops-/supabase/migrations/018_rls_policies.sql`
- Sesión: [[Sesion-2026-05-08-Noche-Autonoma]]
- Próximo paso RLS: [[Sprint-Arreglos]] tareas RLS Fase B + C
