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

## Métricas

- **20 tareas** · **59 horas** · 15 high · 2 medium · 3 low
- **MVP** (binarios firmados descargables): 15 tareas high = **42h ≈ 5 días-hombre**
- **Sprint UUID DB**: `0e26d64e-dd2d-46d5-8278-e78318fba0a0`
- **Tenant**: black-wolf
- **Plazo**: 2026-05-08 → 2026-05-25

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

## Multi-tenant en desktop — problema interesante

La web actual usa URLs `central.blackwolfsec.io/<clientSlug>/...`. En la app desktop **no hay URL bar**. Opciones:

**Opción A — Pantalla de selección al primer login**:
- App fresh → "Selecciona tu organización" → input slug + lista guardada
- Tras login con éxito, guardar `lastSlug` en store
- Próxima vez abre directo en ese slug

**Opción B — App pre-configurada per-tenant**:
- Builds separados con `clientSlug` baked: `BlackWolf-Hugo.dmg`, `BlackWolf-Portillo.dmg`, etc.
- Más fricción para distribuir, pero el alumno ve "su app" no genérica.

**Opción C — Login con email → backend devuelve slug del usuario**:
- Email único en la org → backend resuelve a qué `client_id` pertenece
- App auto-navega a su slug
- Necesita backend endpoint nuevo `/api/auth/resolve-tenant`

Recomendación: **Opción A para MVP, Opción C cuando llegue auth real**. Las apps generic descargables son lo más común (Slack, Discord, Linear todos funcionan así).

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
