---
tags: [tecnologia, db, backup, riesgo]
created: 2026-05-08
updated: 2026-05-08
status: docs
risk: medium-high
---

# Backups Supabase — Dashboard-Ops

Estado real del proyecto Supabase `kyxupfowsfkyqklxhtyo` (Dashboard ops · eu-west-1 · Postgres 17.6.1).

## Estado actual (2026-05-08)

| Característica | Estado | Detalle |
|---|---|---|
| **PITR (Point-In-Time Recovery)** | ❌ **OFF** | Restore solo a snapshot diario, no a timestamp arbitrario. Ventana de pérdida: hasta 24h. |
| **Daily backups** | ✅ ON | Política implícita del plan. 8 backups disponibles, oldest 2026-05-01. |
| **WAL-G** | ✅ ON | Backup engine usado por Supabase. |
| **`pg_dump` automatizado externo** | ❌ NO | Tarea separada del Sprint Arreglos `[db] Backup automatizado pg_dump cron + S3` (4h). |
| **Replicación cross-region** | ❌ NO | El plan actual no la incluye. |

## Riesgo concreto

- Si hay corrupción a las 16:00 hoy, el último backup recuperable es el de hoy 8:00 → **8h de datos perdidos**.
- Para tener PITR (restore a cualquier minuto exacto) hace falta upgrade a tier que lo incluya. Verificar pricing — usualmente +25 USD/mes por proyecto.
- No tenemos copia OFF-Supabase (S3 propio) que sobreviva a un compromise del Supabase project.

## Acciones recomendadas (orden por ROI)

1. **Activar PITR en Supabase** (1 click + ~25 USD/mes). Reduce ventana de pérdida de 24h → 2 min.
2. **Cron `pg_dump` semanal a S3 propio** (Sprint Arreglos `[db]` 4h estimadas). Defensa contra compromise del project Supabase.
3. **Documentar runbook de restore** — qué pasos seguir si hay incidente. Sin esto, en un incidente real perdemos horas decidiendo qué hacer.

## Cómo verificar el estado en cualquier momento

```bash
SUPABASE_MGMT=$(grep -E '^SUPABASE_MANAGEMENT_TOKEN=' /home/blackwolfsec/ejambre/.env | cut -d= -f2)
PROJECT_REF="kyxupfowsfkyqklxhtyo"
curl -s -H "Authorization: Bearer $SUPABASE_MGMT" \
  "https://api.supabase.com/v1/projects/$PROJECT_REF/database/backups" \
  | python3 -m json.tool
```

Salida útil:
- `pitr_enabled` → si `false`, activar.
- `backups[].inserted_at` → días de retención reales.
- `walg_enabled` → siempre `true` para Postgres 14+.

## Conexiones

- Sprint Arreglos: `[db] Backup automatizado pg_dump cron + S3` (4h, medium).
- Sprint Arreglos: `[db] Audit índices (client_id, status) en tablas grandes` (3h, low).
- Auditoría: [[Bugs-Criticos]] no menciona explícitamente PITR off, debería actualizarse.
