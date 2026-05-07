---
tags: [decision-cto, secretos, critical]
status: pending-decision
created: 2026-05-07
---

# Política de Secretos

Decisión pendiente. Hoy todo vive en `.env` que circula y comments del propio `.env` admiten exposure ("rotar al terminar sprint").

## Estado actual (insostenible)

- **Frontend**: secrets en `Dashboard-Ops-/.env` (Vite expone `VITE_*` al cliente — cualquier `VITE_RESEND_KEY` u otro secret ahí leak inmediato)
- **Backend**: secrets en `enjambre-api/.env` (todo el monorepo `ejambre`)
- **Comments admiten exposure**:
  ```
  # rotar tras confirmar conectividad: estas keys circularon en chat
  ```
- **Las keys circularon en chat** y siguen activas en producción

## Riesgo concreto

- 7+ keys exposed: Stripe x2, Resend x2, Anthropic, Supabase service+management, Cloudflare tunnel
- Cualquier ex-colaborador con copia de un `.env` viejo puede:
  - Cobrar via Stripe live de Hugo
  - Mandar mails desde dominios de los tenants
  - Drenar la cuenta Anthropic
  - Acceder a TODOS los datos de Supabase (service_role bypass RLS)

## Opciones a evaluar

### Opción A — Vercel Environment Secrets (frontend)

**Pro**: nativo, integrado con deploy, encryption at rest
**Contra**: solo aplica al frontend. Backend en server propio necesita otra cosa
**Coste**: incluido en plan actual

### Opción B — Doppler (frontend + backend, multi-env)

**Pro**: unifica, sync automático, audit trail, rotation API
**Contra**: dependencia externa, ~$10-20/mes inicio, otro tool en el stack
**Coste**: $10/mes hasta cierto volumen

### Opción C — SOPS + age (backend, file-based encrypted)

**Pro**: gratis, gitops-friendly, sin dependencia de servicio
**Contra**: gestión manual de keys de descifrado, friction
**Coste**: 0

### Opción D — AWS Secrets Manager / GCP Secret Manager

**Pro**: enterprise-grade, audit nativo
**Contra**: dependencia cloud específica, latencia adicional, ~$0.40 por secret/mes
**Coste**: ~$10/mes según volumen

### Opción E — Hashicorp Vault self-hosted

**Pro**: máximo control, rotation automática, dynamic secrets
**Contra**: setup pesado, mantenimiento, overkill para tamaño actual
**Coste**: 0 + tiempo

## Recomendación inicial

**Híbrido**:
- **Vercel Environment Secrets** para Dashboard-Ops (gratis, nativo)
- **Doppler** para enjambre-api (centralizo backend + scripts + Donna)

Es el mínimo común para tener:
- Audit trail de quién accede a qué secret
- Rotación atómica (cambias en Doppler → live en backend en segundos)
- Separación clara de envs (dev / staging / prod)
- Sin keys en `.env` del repo (ni tu local ni copia ajena)

## Tareas si se aprueba la recomendación

1. Crear cuenta Doppler + project para `enjambre-api`
2. Importar `.env` actual a Doppler
3. Configurar `enjambre-api/src/server.js` para leer de `process.env` (sin cambios — Doppler inyecta env vars)
4. Cambiar startup a `doppler run -- node src/server.js` o usar Doppler CLI
5. Migrar Vercel: `Dashboard-Ops-/.env` → Vercel dashboard env vars (con `VITE_*` para los públicos)
6. **Borrar `.env` del repo + del filesystem local**
7. **Rotar TODAS las keys** (algunas ya están exposed)
8. Documentar runbook de rotación en [[Playbook-Rotar-Secret]]

## Quién decide

Tú (CTO). No tiene sentido que tome esta decisión un junior — implica integrar un servicio externo o decidir mantenerlo file-based.

## Conexiones

- Sangrando: [[Bugs-Criticos#C1]]
- Cierra: [[Sprint-0-Higiene]] (parche) · [[Sprint-4-Webhooks-Secrets]] (estructural)
- Procedimiento: [[Playbook-Rotar-Secret]]
