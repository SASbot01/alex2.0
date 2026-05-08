---
tags: [integraciones, meta, runbook]
created: 2026-05-08
updated: 2026-05-08
trigger: cron alert `connector.meta_token_expiring` o `meta_token_invalid`
---

# Meta CAPI Token Refresh — Runbook

## Cuándo activar

Cuando uno de estos signals dispare:

1. **Cron `/api/cron/check-meta-token`** publica `meta_token_expiring` (días < 14) o `meta_token_invalid`.
2. **Logs**: errores `OAuthException` o `190` desde llamadas a `https://graph.facebook.com/v22.0/<pixel_id>/events`.
3. **Síntoma silencioso**: caída repentina de eventos CAPI en Meta Events Manager (verificar dashboard).

## Pre-requisitos

- Acceso al Meta Business Manager con permisos sobre la app (BlackWolf / Asesoría Suiza).
- `META_APP_ID` y `META_APP_SECRET` (Vercel env vars o copiados desde la app config en Meta).
- El token actual `META_CAPI_TOKEN` (Vercel env var).

## Procedimiento (manual, 10 min)

### Paso 1 — Generar nuevo long-lived token

Si el token actual aún es válido pero próximo a expirar, intercambiarlo por uno nuevo:

```bash
APP_ID="<META_APP_ID>"
APP_SECRET="<META_APP_SECRET>"
CURRENT_TOKEN="<META_CAPI_TOKEN>"

curl "https://graph.facebook.com/v22.0/oauth/access_token?grant_type=fb_exchange_token&client_id=${APP_ID}&client_secret=${APP_SECRET}&fb_exchange_token=${CURRENT_TOKEN}"
```

Respuesta:
```json
{
  "access_token": "EAA...nueva...",
  "token_type": "bearer"
}
```

Si el token está **ya expirado** o invalidado, el endpoint devuelve `OAuthException` 190. En ese caso hay que generar manualmente desde Meta Graph API Explorer o System User token (preferible para CAPI):

1. Meta Business Manager → Business Settings → Users → System Users.
2. Crear/editar System User → "Generate New Token".
3. Selecciona los assets (App, Pixel) y permisos: `ads_management`, `business_management`.
4. Token tipo "Never expires" si es System User token.

### Paso 2 — Verificar el nuevo token

```bash
curl "https://graph.facebook.com/v22.0/debug_token?input_token=${NEW_TOKEN}&access_token=${APP_ID}|${APP_SECRET}"
```

Verificar:
- `is_valid: true`
- `expires_at: 0` (System User) o `expires_at` > 60 días en el futuro (long-lived user token).
- `scopes` incluye `ads_management` y/o `business_management`.

### Paso 3 — Actualizar Vercel env var

```bash
VERCEL_TOKEN=$(grep -E '^VERCEL_TOKEN=' /home/blackwolfsec/ejambre/.env | cut -d= -f2)
PRJ="prj_fD0OAHz31ZoyPuO8GJVLe9yXeNY7"  # Dashboard-Ops project ID

# Encontrar el ID del env var existente
ENV_ID=$(curl -s -H "Authorization: Bearer $VERCEL_TOKEN" "https://api.vercel.com/v9/projects/$PRJ/env" | python3 -c "
import json, sys
d = json.load(sys.stdin)
for e in d.get('envs', []):
    if e.get('key') == 'META_CAPI_TOKEN':
        print(e.get('id')); break
")

# Update el value
curl -s -X PATCH -H "Authorization: Bearer $VERCEL_TOKEN" -H "Content-Type: application/json" \
  "https://api.vercel.com/v9/projects/$PRJ/env/$ENV_ID" \
  -d "{\"value\":\"${NEW_TOKEN}\",\"target\":[\"production\",\"preview\",\"development\"]}"
```

### Paso 4 — Trigger redeploy

Vercel envs aplican al SIGUIENTE deploy. Para forzar:

```bash
# Re-deploy production con el current main
curl -X POST -H "Authorization: Bearer $VERCEL_TOKEN" -H "Content-Type: application/json" \
  "https://api.vercel.com/v13/deployments" \
  -d "{\"name\":\"dashboard-ops\",\"target\":\"production\",\"gitSource\":{\"type\":\"github\",\"repo\":\"aatshadow/Dashboard-Ops-\",\"ref\":\"main\"}}"
```

O simplemente push un commit trivial a `main` y deja que Vercel deploy automático lo haga.

### Paso 5 — Verificar en producción

```bash
# Trigger el cron manualmente para confirmar
curl -X POST https://central.blackwolfsec.io/api/cron/check-meta-token \
  -H "Authorization: Bearer ${CRON_SECRET}"
# Esperado: { ok: true, is_valid: true, days_to_expire: 60 }
```

Y enviar un evento de prueba al pixel:

```bash
PIXEL_ID="<META_PIXEL_ID>"
curl -X POST "https://graph.facebook.com/v22.0/${PIXEL_ID}/events?access_token=${NEW_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "data":[{
      "event_name":"PageView",
      "event_time":'$(date +%s)',
      "user_data":{"em":["test@example.com"]},
      "test_event_code":"TEST123"
    }]
  }'
# Esperado: { events_received: 1, ... }
```

Verificar en Meta Events Manager → Test Events que llegó.

### Paso 6 — Documentar

- Anotar fecha de refresh + nueva expiración en este doc.
- Si fue System User token (never-expires), eliminar este runbook (deja de aplicar).

## Histórico

| Fecha | Tipo de token | Próxima expiración | Notas |
|---|---|---|---|
| 2026-05-08 | (pendiente verificar tipo actual) | desconocido | Cron `check-meta-token` añadido por Sprint Arreglos. |

## Refresh automático (futuro)

Cuando `META_APP_ID` y `META_APP_SECRET` estén en `.env`/Vercel, el cron puede:
1. Detectar `days_to_expire < 7`.
2. Hacer el `/oauth/access_token` exchange.
3. Actualizar Vercel env var via API (paso 3 automatizado).
4. Trigger redeploy (paso 4 automatizado).

Esto reduce la operación manual a 0. Tarea futura: `[integraciones] Meta Ads token refresh full automation` (3-4h adicional al monitoring actual).

## Conexiones

- Cron: `Dashboard-Ops-/api/cron/check-meta-token.js`
- Schedule: `vercel.json` → `0 9 * * 1` (lunes 9:00 UTC).
- Eventos publicados (potencial): `connector.meta_token_expiring`, `connector.meta_token_invalid`.
