---
tags: [playbook, secretos]
created: 2026-05-07
---

# Playbook — Rotar un Secret

Procedimiento estándar cuando una API key se ha expuesto, se sospecha exposure, o periódicamente cada 90 días.

## Cuándo

- ✅ Cualquier sospecha de exposure (chat, screenshot, repo público, ex-empleado)
- ✅ Detección de uso anómalo (consumo inesperado, IP rara)
- ✅ Periódicamente cada 90 días aunque no haya señal

## Procedimiento general (4 pasos)

### 1. Generar la nueva key

Cada servicio tiene su flow. Ver sección por servicio abajo.

### 2. Hacer la nueva activa SIN borrar la vieja

Esto evita downtime durante el switch. Algunas plataformas permiten 2 keys simultáneas — usa esto.

### 3. Actualizar todos los consumers

Por servicio: ver "donde vive cada secret" abajo.

### 4. Revocar la vieja + verificar

`curl` con la key vieja → debe devolver 401.

---

## Donde vive cada secret (mapping completo)

```
ANTHROPIC_API_KEY
  ├─ enjambre-api/.env (orchestrator + commanders + workers)
  └─ Dashboard-Ops-/.env (cv-coach-suiza, agent.js, otros)

STRIPE_SECRET_KEY_HUGO     (rk_live_...)
STRIPE_WEBHOOK_SECRET_HUGO (whsec_...)  ← NO CONFIGURADO HOY (ver C2)
  └─ enjambre-api/.env

STRIPE_SECRET_KEY_PORTILLO (rk_live_...)
STRIPE_WEBHOOK_SECRET_PORTILLO (whsec_...)
  └─ enjambre-api/.env

RESEND_API_KEY (multi-tenant via DB)
  └─ Tabla email_config (per-tenant, account_index)
  └─ enjambre-api/.env (RESEND_API_KEY default global, fallback)
  └─ Dashboard-Ops-/.env (CV forwarding asesoria-suiza)

SUPABASE_SERVICE_KEY
SUPABASE_MANAGEMENT_TOKEN
  ├─ enjambre-api/.env
  └─ /home/blackwolfsec/ejambre/.env (root)

VITE_SUPABASE_ANON_KEY
  └─ Dashboard-Ops-/.env (público — Vite expone, OK que se vea)

CLOUDFLARE_TUNNEL_TOKEN
  └─ /root/.cloudflared/<config> (systemd service)
  └─ /home/blackwolfsec/ejambre/.env (eyJ...)

PORTAL_JWT_SECRET (nuevo, post-Hito 2 Portal)
  └─ enjambre-api/.env (PENDIENTE setear)

JWT_SECRET (auth de team)
  └─ enjambre-api/.env

GOOGLE_CLIENT_SECRET
GOOGLE_DRIVE_REFRESH_TOKEN
  └─ Dashboard-Ops-/.env (server-side API routes)

DONA_PEER_TOKEN
  └─ enjambre-api/.env (army general daemon)
  └─ Donna /opt/ai-agent-console (consumer)
```

---

## Por servicio — pasos detallados

### Anthropic

1. console.anthropic.com → API Keys
2. Create new key
3. Copy → meter en `enjambre-api/.env` y `Dashboard-Ops-/.env`
4. Reiniciar enjambre-api
5. Vercel redeploy frontend (o `vercel env pull` + redeploy)
6. Test: una llamada a Claude desde cada componente
7. Revoke old key
8. Validar: old key → 401

### Stripe (cuenta del tenant)

1. Stripe Dashboard de la cuenta del tenant → Developers → API keys
2. **Restricted key** preferido sobre secret. Permisos mínimos: products, prices, charges, customers, payment_intents, checkout_sessions, subscriptions
3. Copy → meter en `enjambre-api/.env` con naming `STRIPE_SECRET_KEY_<TENANT>`
4. Reiniciar enjambre-api
5. Test: hacer venta de prueba (test mode) o consultar `GET /v1/customers`
6. Revoke old key
7. Validar

**Nota**: cuando rotas, tienes que reconfirmar **webhook secret** sigue válido. La rotación de signing secret de webhook es separada (Webhooks → endpoint → "Signing secret" → "Roll").

### Resend

Si es la global (`RESEND_API_KEY`):
1. Resend dashboard → API Keys → revoke old + create new
2. Update `enjambre-api/.env` y `Dashboard-Ops-/.env`
3. Reiniciar consumers

Si es la per-tenant (en DB `email_config`):
1. Resend dashboard → API Keys (de la cuenta del tenant)
2. Revoke + create new
3. UPDATE en DB:
   ```sql
   UPDATE email_config 
   SET api_key = 'new_key' 
   WHERE client_id = '<uuid>' AND account_index = 1;
   ```

### Supabase

**Service role**:
1. Supabase Dashboard → Project Settings → API → service_role key → "Reset"
2. ⚠️ Esto invalida la key inmediatamente — preparar el nuevo deploy antes de pulsar
3. Update `enjambre-api/.env`
4. Reiniciar enjambre-api

**Management token** (más sensible — usado por scripts):
1. Account → Access Tokens → revoke old + generate new
2. Update `/home/blackwolfsec/ejambre/.env`

### Cloudflare Tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → enjambre-v2 → Configure
2. Refresh token (UI)
3. Update `/root/.cloudflared/<config>.yml` o env
4. `systemctl restart cloudflared-enjambre.service`

### JWT_SECRET / PORTAL_JWT_SECRET

⚠️ **Romper sesiones activas es ESPERADO al rotar JWT_SECRET**. Todo team logueado tendrá que volver a iniciar sesión. Anunciar antes.

```bash
# Generar nueva
openssl rand -hex 64

# Actualizar
vim enjambre-api/.env
# JWT_SECRET=<nueva>

# Restart
docker compose restart enjambre-api  # o systemctl, según deploy
```

Para PORTAL_JWT_SECRET el efecto es: todos los alumnos del portal tienen que volver a pedir OTP. Aviso por mail recomendable.

---

## Post-rotación (siempre)

- [ ] Validar que old keys → 401 contra cada servicio
- [ ] Loguear en [[Diario-Decisiones]]: fecha, motivo, qué se rotó
- [ ] Actualizar [[Sesion-rotacion-secrets-YYYY-MM-DD]] si es trabajo grande
- [ ] Si la exposición fue por chat/screenshot, **borrar el chat/screenshot también**

## Conexiones

- Política: [[Politica-de-Secretos]]
- Sangrando hoy: [[Bugs-Criticos#C1]]
- Sprint que cierra: [[Sprint-0-Higiene]]
