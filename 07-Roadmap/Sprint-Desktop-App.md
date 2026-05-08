---
tags: [sprint, desktop, tauri, dashboard-ops]
sprint: desktop-app
duration: ~2 weeks
status: active
created: 2026-05-08
start_date: 2026-05-08
end_date: 2026-05-25
---

# Sprint Desktop App — Dashboard-Ops nativa Mac + Windows (Tauri)

Convertir [[Dashboard-Ops]] en app de escritorio descargable usando **Tauri**. Cero refactor de la SPA actual — Tauri es un wrapper alrededor del build de Vite con un WebView nativo del SO.

## Métricas (post-decisiones 2026-05-08)

- **23 tareas** · **74.5 horas** · 13 high · 4 medium · 6 low
- **MVP sin firmas** (binarios descargables, abrir con "click derecho > Abrir igualmente" Mac + "Run anyway" Win): **44h high ≈ 5.5 días-hombre**
- **Sprint UUID DB**: `0e26d64e-dd2d-46d5-8278-e78318fba0a0`
- **Tenant**: black-wolf
- **Plazo**: 2026-05-08 → 2026-05-25

## Decisiones tomadas (2026-05-08)

| Decisión | Elección | Implica |
|---|---|---|
| Apple Developer Account | **Diferido** | MVP sin firma Mac. Usuario Mac: "click derecho → Abrir → Abrir igualmente" la primera vez |
| Win code-signing cert | **Diferido** | MVP sin firma Win. Usuario Win: SmartScreen "More info → Run anyway" la primera vez |
| Multi-tenant | **Opción B** — builds separados per-tenant | `Hugo-Dashboard.dmg`, `Portillo-Dashboard.dmg`, etc. Cada tenant ve su marca |
| Versión inicial | `v0.1.0-prerelease` | Sin SLA, etiqueta de no producción |

**Impacto**: ahorro ~600-700€/año en certs + ~15h de tareas de signing. MVP arrancable en ~5.5 días-hombre. Cuando se distribuya externamente o se quiera UX limpia, se compran los certs y se activan las 6 tareas en `low` (Mac signing + Win signing + Pre-requisitos + App Stores + Optimización).

## Por qué Tauri y no Electron

| Métrica | Tauri ⭐ | Electron |
|---|---|---|
| Peso binario final | 3-8 MB | 80-120 MB |
| Lenguaje wrapper | Rust | Node.js |
| WebView | Nativo del SO (WKWebView Mac, WebView2 Win) | Chromium completo embebido |
| Memoria al correr | ~50-80 MB | ~200-400 MB |
| Auto-updater oficial | ✅ plugin Tauri | ✅ electron-builder |
| Refactor del React | 0 | 0 |
| Curva aprendizaje Rust | Pequeña (solo config) | N/A |

Decisión registrada — coincide con stack moderno y con vibe técnico del CTO.

## Pre-requisitos comprables (PRIMERO)

| Item | Coste | Para qué |
|---|---|---|
| Apple Developer Account | ~99 €/año | Firmar + notarizar Mac. Sin esto, Mac muestra "App de desarrollador no identificado" y muchos usuarios no abren. |
| Win code-signing cert (OV) | 200-300 €/año | Firma básica Windows. SmartScreen tarda semanas en confiar mientras gana reputación. |
| Win code-signing cert (EV) | 400-600 €/año | SmartScreen verde de día 1. Recomendable para producto serio. |
| Dominio para auto-update | 0 (ya tienes) | `blackwolfsec.io/desktop/latest.json` |

**La entrega del cert puede tardar 1-3 días hábiles**. Compra YA si vas a empezar el sprint, no esperes a estar al final del build.

## Tareas top — orden de ejecución

### Fase 1: Setup (8h, ~1 día)

1. Pre-requisitos: comprar Apple Developer + Win cert (1h gestión)
2. Instalar Rust + tooling Tauri en local (1h)
3. Init Tauri sobre repo Dashboard-Ops (4h)
4. Configurar `tauri.conf.json` (window, identifiers, permisos) (2h)

### Fase 2: Build basic (8h, ~1 día)

5. Crear iconos `.icns` / `.ico` / `.png` desde logo BlackWolf (2h)
6. CORS backend whitelist Tauri origin (1h)
7. Multi-tenant: pantalla selección de slug (3h)
8. Decidir flujo auth desktop (deep-link vs embedded) (4h — incluye prototipo)

### Fase 3: Auth + tokens seguros (4h, ~0.5 día)

9. Migrar token de localStorage a `tauri-plugin-store` (4h)

### Fase 4: Firma + release (15h, ~2 días)

10. Firma + notarización Mac (4h)
11. Firma Windows (3h)
12. Auto-updater Tauri (4h)
13. CI GitHub Actions runners macOS + Windows (4h)

### Fase 5: Distribución (5h)

14. Página `/download` en Dashboard-Ops con detección OS (3h)
15. Smoke test Mac fresh + Win fresh (2h)

### Fase 6: Nice to have (post-MVP, ~17h)

16. Notificaciones nativas (3h)
17. Tray icon con menú contextual (4h)
18. Optimización bundle (2h)
19. Mac App Store (4h)
20. Microsoft Store (4h)

---

## Adaptaciones que hay que hacer en el código

Lo bueno: **muy pocas**. La SPA sigue funcionando idéntica.

Lo que SÍ cambia:
- **`src/utils/portalAuth.js`** y similares: detectar entorno Tauri (`window.__TAURI_INTERNALS__`) y usar `tauri-plugin-store` en lugar de localStorage. Token encriptado por el SO (Keychain Mac, Credential Manager Win).
- **`enjambre-api/src/server.js`**: añadir `tauri://localhost` y `https://tauri.localhost` al whitelist CORS según versión.
- **`vite.config.js`**: posiblemente añadir `clearScreen: false` para no perder logs en `tauri dev`. Cambios mínimos.
- **`Dashboard-Ops/src/main.jsx`**: añadir manejo de URL inicial (la app abre en `index.html`, no en `central.blackwolfsec.io/login` automáticamente).
- **API URL absoluto**: la app desktop sigue hablando con `central.blackwolfsec.io` y `enjambre.blackwolfsec.io` por HTTPS. Los datos viven igual en Supabase + enjambre-api.

## Multi-tenant en desktop — Opción B (decidida 2026-05-08)

**Decisión**: builds separados per-tenant. Cada cliente ve SU app con SU marca.

```
Hugo-Dashboard-Mac-Universal.dmg     → app productName "Hugo Dashboard", icon Hugo
Portillo-Dashboard-Mac-Universal.dmg → "Asesoría Suiza Dashboard", icon Asesoría Suiza
FBA-Academy-Dashboard-Win-x64.exe    → "FBA Academy Dashboard", icon FBA
...
```

### Implementación

1. **`src-tauri/tenants.json`** centraliza metadata per-tenant:
   ```json
   {
     "enformaconhugo": {
       "productName": "Hugo Dashboard",
       "identifier": "io.blackwolfsec.dashboard-ops.enformaconhugo",
       "icon": "icons/hugo/icon.icns",
       "primaryColor": "#XXXXXX",
       "loginUrl": "https://central.blackwolfsec.io/enformaconhugo/login"
     },
     "asesorias-suiza": { ... }
   }
   ```

2. **Build time**: variable env `TENANT_SLUG=enformaconhugo` lee `tenants.json` y patchea `tauri.conf.json` antes de `cargo tauri build`.

3. **CI matrix** en GitHub Actions:
   ```yaml
   strategy:
     matrix:
       tenant: [enformaconhugo, asesorias-suiza, fba-academy, yc-logistics, creator-founder, detras-de-camara]
       os: [macos-latest, windows-latest]
   ```
   = 6 tenants × 2 OS = 12 builds por release. Tiempo estimado con cache cargo: ~10-15 min total.

4. **Constants embebidos**: `src/constants/tenantConfig.js` se genera al build con el slug correcto. Login form pre-llenado, no muestra picker.

5. **Naming convention** binarios: `<Tenant>-Dashboard-<OS>-<arch>.<ext>`.

### Trade-offs vs Opción A (slug picker)

✅ Cliente ve SU marca, sensación de producto dedicado
✅ Login pre-llenado, 0 fricción tras descarga
✅ Posibilidad futura de pricing tier (build con features extra para tenant premium)
❌ N artefactos por release (vs 1) — gestionable con CI matrix
❌ Cuando llegue cliente nuevo, hay que añadir slug a `tenants.json` y rebuildar
❌ Si onboardas 50 clientes, 100 binarios por release — buscar paths filters por tenant cambiado

## Sin firmar — UX del primer abrir

### Mac
La primera vez que el usuario abre el `.dmg`, Gatekeeper bloquea con "App de desarrollador no identificado".

**Workaround**:
1. **Click derecho sobre la app** (no doble-click)
2. **Abrir** desde el menú contextual
3. Diálogo de confirmación → **Abrir**
4. La siguiente vez, ya se abre normal con doble-click

Documentar en banner de `/download`.

### Windows
SmartScreen muestra "Windows protegió tu PC".

**Workaround**:
1. Click en **More info** (texto pequeño, fácil de no ver)
2. Aparece botón **Run anyway**
3. Click → instala normal

Documentar en banner de `/download`.

### Cuándo comprar los certs

Comprar **antes** de:
- Distribuir a usuarios fuera del círculo de confianza interno
- Soporte técnico se queja de "muchos usuarios reportan que no saben abrir la app"
- Onboarding masivo de clientes nuevos

Las 3 tareas en `low` (Mac signing + Win signing + Pre-requisitos compra) se reactivan a high cuando llegue ese momento.

## Distribución (cómo lo descarga la gente)

### Página `/download`

```
central.blackwolfsec.io/download
  ↓
Detecta OS del visitante:
  - Mac (Apple Silicon)  → BlackWolf-Mac-Universal.dmg
  - Mac (Intel)          → BlackWolf-Mac-x64.dmg
  - Windows              → BlackWolf-Win-Setup.exe
  - Linux (no priorizado) → BlackWolf-Linux.AppImage
```

### Auto-update flow

```
1. App al boot → fetch blackwolfsec.io/desktop/latest.json
2. Compara version del manifest con package.json local
3. Si manifest > local → notif "New version available"
4. User acepta → descarga signed update package
5. Verifica firma con public key
6. Instala + reinicia app
```

## Riesgos / decisiones pendientes

- **Distribución sin App Store**: SmartScreen Windows tarda 2-4 semanas en "confiar" en certs nuevos OV. EV cert lo soluciona. Decidir si gastar la diferencia (~200€/año) por la mejor UX inicial.
- **Notarización Mac**: requiere Internet en build time. Si CI runner no tiene credentials Apple, falla. Configurar bien los secrets de GitHub Actions.
- **Versiones**: Tauri estable es v2.x ahora. Romper compat entre v1 y v2 era heavy. Empezar directo en v2 para no migrar.
- **Self-hosted updater vs proveedor**: hoy montamos endpoint propio. Alternativas: Tauri Updater plan, GitHub Releases CDN. Empezar self-hosted (control total), migrar si dolor.

## Conexiones

- Producto: [[Dashboard-Ops]]
- Stack: [[Frontend-React-Vite]] (no cambia) + nuevo Tauri/Rust en `src-tauri/`
- Auth deuda: [[Bugs-Criticos#C5]] (localStorage `bw_superadmin`) — la migración a httpOnly cookies + tauri-plugin-store es el momento idóneo de cerrar también este bug
- Política de secretos: [[Politica-de-Secretos]] (los signing keys del updater hay que guardarlos en el secret manager elegido)
- Sprint hermano: [[Sprint-Setter-WhatsApp]] · [[Sprint-Arreglos]]
