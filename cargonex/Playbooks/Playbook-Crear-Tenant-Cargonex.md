---
tags: [playbook, cargonex, onboarding]
created: 2026-05-07
---

# Playbook — Crear Tenant Cargonex

Procedimiento para convertir un lead de [[Cargonex]] en un tenant productivo. Distinto del [[Playbook-Onboarding-Cliente-Nuevo]] (ese es para tenants de [[Dashboard-Ops]]).

## Cuándo

- Lead aprobado en `/admin → Leads` (status: `new` → `contacted` → `converted`).
- Cliente firmó contrato + DPA si aplica DSGVO (cliente alemán/UE).
- Tienes el nombre legal de la empresa, email de contacto, plan acordado.

## Fase 0 — Decisiones previas

- [ ] **Slug**: nombre CamelCase corto que vaya bien en URL. Ejemplos: `EilersLogistik`, `MuellerSpedition`, `AcmeGmbH`. Validar que no choca con palabras reservadas (`admin`, `portal`, `landing`, `api`, etc.).
- [ ] **Plan**: `starter` · `professional` · `enterprise`. Define límites (no enforced hoy, solo display).
- [ ] **Idiomas operativos**: DE/EN/ES — si nuevo idioma, planificar i18n.
- [ ] **Email contacto**: el del director, no soporte genérico.
- [ ] **DSGVO aplicable**: si UE → DPA firmado antes de tocar nada productivo.

## Fase 1 — Crear el tenant en Cargonex

Vía panel:

1. Login en `https://app.cargonex.co/admin` con `PLATFORM_ADMIN_SECRET`.
2. **Clients → + New client**.
3. Rellenar: company name, contact email, plan, slug.
4. Confirmar.

Vía curl (equivalente, útil para scripts):

```bash
SECRET=$(grep '^PLATFORM_ADMIN_SECRET=' /home/s4sf/eilerrs-portal/.env | cut -d= -f2-)
curl -s -X POST https://app.cargonex.co/api/platform-admin \
  -H 'Content-Type: application/json' \
  -H "X-Admin-Secret: $SECRET" \
  -d '{
    "action": "create-client",
    "companyName": "Acme GmbH",
    "contactEmail": "ceo@acme.de",
    "plan": "professional",
    "slug": "AcmeGmbH"
  }'
```

Respuesta:

```json
{
  "ok": true,
  "clientId": "cli_xyz123",
  "key": "CX-AB12CD34-EF56",
  "slug": "AcmeGmbH",
  "status": "pending",
  "createdAt": "2026-05-07T…Z"
}
```

**Apuntar el `key` y el `clientId`** — el key NO se vuelve a mostrar en plano (solo el último-4 hash). Si lo pierdes, hay que re-emitirlo (`revoke-key` + `issue-key`).

## Fase 2 — Mandar credenciales al cliente

Email/WhatsApp con dos cosas:

```
Hola [Cliente],

Tu portal Cargonex está listo. Para activarlo:

1. Ve a https://app.cargonex.co/portal
2. Pulsa "Erstes Mal? PI-Schlüssel verwenden"
3. Introduce tu PI-Key:  CX-AB12CD34-EF56
4. Tu portal aterrizará en https://app.cargonex.co/AcmeGmbH

Después del primer uso, el siguiente login es con email + contraseña que
hayas elegido (o email-OTP si prefieres no usar password).

Cualquier problema, respondes a este email.
```

Adaptar idioma según cliente (DE para alemanes).

## Fase 3 — Cliente activa

Lo hace el cliente solo:

- Va a `/portal` → click "Erstes Mal? PI-Schlüssel verwenden" → introduce key.
- Backend (`handleActivateClient`):
  - Match key vs `cargonex-platform.json#keys`.
  - Flip `tenant.status: pending → active`.
  - Crear `data/tenant-<clientId>.json` (vacío).
  - Escribir `audit` entry.
- Redirect a `/<Slug>`.

Verificar desde tu lado:

```bash
curl -s -X POST https://app.cargonex.co/api/platform-admin \
  -H "X-Admin-Secret: $SECRET" \
  -H 'Content-Type: application/json' \
  -d '{"action":"list-clients-with-stats"}' | jq '.clients[] | select(.slug=="AcmeGmbH")'
```

`status` debe ser `active` y `activatedAt` no nulo.

## Fase 4 — Configurar SMTP per-tenant (si aplica)

Si quieres que los emails transaccionales del cliente salgan de su dominio:

- [ ] Verificar dominio del cliente en Resend (panel BlackWolf)
- [ ] Generar API key per-tenant
- [ ] (Hoy NO hay UI per-tenant para esto en Cargonex) — se mete a mano en `data/tenant-<clientId>.json#emailConfig` o se queda con el SMTP global del stack.

> **Gap**: Cargonex hoy usa un único SMTP_* global. No hay aislamiento per-tenant para emails como en [[Dashboard-Ops]] (`email_config` con `account_index`). Si lo necesitas, plantear.

## Fase 5 — Validación E2E

- [ ] Login del director en `/<Slug>` → ver dashboard
- [ ] Crear un driver de prueba → confirmar persiste tras refresh
- [ ] Crear un vehicle de prueba → confirmar
- [ ] Generar un record (Schicht/turno) → confirmar
- [ ] Sincronizar (botón "Sync") → confirmar `/api/tenant-logs` returns 200
- [ ] Si web-push aplicable: subscribirse → recibir test notification
- [ ] Logout → login otra vez con email+password → confirmar funciona
- [ ] Borrar los datos de prueba

## Fase 6 — Anotar en el cerebro

- [ ] Crear `02-Clientes/<NombreCliente>.md` siguiendo [[Eilers-Logistik]] como template
- [ ] Añadir fila a [[_moc/MOC-Clientes]]
- [ ] Si hay ADR específico (multi-owner, cumplimiento extra), añadir en `08-Decisiones-CTO/`

## Gotchas aprendidos

- **Slugs case-sensitive** — `eilersLogistik` y `EilersLogistik` son tenants distintos. Decide la convención y stick to it. Cargonex usa CamelCase.
- **Keys son one-shot display** — perderlo significa rotación. Decir esto al cliente al mandar el email.
- **`pending` → 302 a /portal** — si el cliente entra a `/<Slug>` ANTES de activar, le rebotamos a `/portal`. Si abusan/se confunden, mandar de nuevo el link de activación, no el del slug.
- **Eilers es legacy** — cualquier acción "list all tenants and bulk-do X" debe contemplar que Eilers vive en `db.json` y los demás en `tenant-<id>.json`. Hay que llamar a `getTenantDataFile()`, no asumir paths.
- **SW cache** — si el cliente reporta UI vieja después de un deploy, hard-refresh + bump de `SW_VERSION` si llevas tiempo sin bumpear.

## Revocar un tenant

Cuando un cliente cancela:

```bash
curl -s -X POST https://app.cargonex.co/api/platform-admin \
  -H "X-Admin-Secret: $SECRET" \
  -H 'Content-Type: application/json' \
  -d '{"action":"revoke-client","clientId":"cli_xyz123","reason":"Customer cancelled"}'
```

Efecto: `status: revoked`. `/<Slug>` devuelve 403. El data file `tenant-<clientId>.json` queda en disco (data retention según contrato/DSGVO). Borrado físico aparte.

## Conexiones

- Producto: [[Cargonex]] · [[Arquitectura]]
- Tenant fundador (template): [[Eilers-Logistik]]
- Distinción con onboarding Dashboard-Ops: [[Playbook-Onboarding-Cliente-Nuevo]]
- Si toca rotar secret: [[Playbook-Rotar-Secret]]
