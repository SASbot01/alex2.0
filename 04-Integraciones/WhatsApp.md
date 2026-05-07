---
tags: [integracion, whatsapp, critical]
created: 2026-05-07
---

# WhatsApp — whatsapp-web.js

**Estado**: 🔴 LIVE pero estructuralmente frágil. Plan B inexistente. Banneo del 26-abr en Creator Founder fue el primer aviso.

## Setup actual

- **Librería**: `whatsapp-web.js` v1.34.6 (NO oficial, simula cliente web)
- **Multi-sesión**: hasta 5 sesiones simultáneas (`WA_MAX_SESSIONS`)
- **Storage**: `/app/.wwebjs_auth/<sessionKey>/` por cliente:account
- **Auth**: QR-based (escanear con móvil del owner)
- **Backend**: `enjambre-api/src/connectors/whatsapp.js` (1427 LOC — necesita refactor)

## Flujo

```
1. Cliente solicita conectar nueva sesión
2. Backend genera QR
3. Owner escanea con su WhatsApp móvil
4. Sesión persiste en disco
5. Mensajes entrantes:
   - findOrCreateCrmContact (race condition pendiente fix)
   - Tagear con whatsapp_owner (multi-owner) y whatsapp_account_index
   - INSERT crm_messages
   - eventBus 'whatsapp.message.received'
   - (opcional) Setter AI responde via orchestrator
```

## Health check

`whatsapp.js` expone state machine:
- `initStartedAt`
- `lastQrAt`
- `lastStateChangeAt`
- `client.isReady`

Frontend infiere:
- "arrancando" si <5min desde init
- "esperando QR" si lastQrAt reciente
- "ready" si isReady
- "colgado" si >10min sin transición

## Problemas conocidos

### 1. Banneo (incidente 26-abr Creator Founder)

WhatsApp baneó el número durante outreach masivo. La detección anti-bot de WhatsApp es agresiva con clientes web no oficiales. **Sin recovery**: cuando pasa, hay que conseguir otro número y reescanear.

### 2. Race condition `findOrCreateCrmContact`

Dos sesiones simultáneas (multi-owner) creando contactos para el mismo número → duplicados. Ya pasó. Fix pendiente: UNIQUE constraint + `ON CONFLICT`. Ver [[Bugs-Criticos#C8]].

### 3. Sesiones en disco sin encriptar

`.wwebjs_auth/<sessionKey>` plaintext. Acceso de root al server = acceso a la cuenta WhatsApp.

### 4. Memoria — Chromium puppeteer x N sesiones

Cada sesión = Chromium headless = ~50-100 MB. 5 sesiones = ~300-500 MB. Sin alarmas si OOM.

### 5. Sin detección de "account banned"

Si número baneado, sesión sigue reintentar indefinidamente. Frontend espera QR que nunca llega. UX confusa.

## Decisión pendiente — migrar a WhatsApp Business API

[[Decision-WhatsApp]]: 3 opciones evaluadas:

- **360dialog** — BSP popular, pricing por conversación
- **Twilio** — más caro, mejor soporte enterprise
- **Meta directo** — barato pero más burocracia

Estimación migración: 4-6 semanas. Ver [[Sprint-6-WhatsApp-BA]].

**Adapter intermedio** propuesto: capa en backend que abstraiga `whatsapp-web.js` vs WhatsApp Business API → migración cliente por cliente sin downtime.

## Mientras tanto (mitigaciones)

- Limitar a **3 sesiones simultáneas** en producción
- Implementar detección de banneo (timeout persistente sin QR)
- Encriptar `.wwebjs_auth` con `crypto-js` o Node crypto
- Monitor memoria por sesión, auto-kill si >200MB

## Configuración env

```env
# enjambre-api/.env
WA_MAX_SESSIONS=5
WA_DEBOUNCE_MS=10000
WA_RATE_LIMIT_PER_MIN=20
```

## Conexiones

- Bug: [[Bugs-Criticos#C8]] · [[Bugs-Criticos#C9]]
- Decisión: [[Decision-WhatsApp]]
- Sprint: [[Sprint-6-WhatsApp-BA]]
- Cliente afectado: [[Creator-Founder]] (banneo 26-abr)
- Backend code: `enjambre-api/src/connectors/whatsapp.js`
