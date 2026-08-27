---
name: docs-alignment
description: Use when a change affects architecture, stack, scope, or conventions in an octalitycl repo. Keeps AGENTS.md and related docs in sync with the actual code.
license: MIT
---

# Docs Alignment — octalitycl

## Fuente de verdad (que vive donde)

| Doc | Contenido |
|---|---|
| `AGENTS.md` | Stack, decisiones arquitectonicas, fuera de alcance (no-goals), convenciones, flujo de trabajo |
| `SECURITY.md` | Modelo de amenaza, como reportar una vulnerabilidad |
| `.github/CONTRIBUTING.md` | Como se trabaja en GitHub (branches, PR, review) |
| `README.md` | Que es el proyecto + como levantarlo local |

<!-- Si el proyecto es grande y tiene docs/ dedicada (ver PLAYBOOK.md de
     octalitycl/config, Decision 9), agregar aca las rutas reales:
     docs/ARCHITECTURE.md, docs/DATABASE_MODEL.md, etc. -->

## Regla cambio -> doc

| Cambio | Doc a actualizar |
|---|---|
| Cambio de stack (libreria, runtime, proveedor de deploy) | `AGENTS.md` (Stack) |
| Decision tecnica significativa (por que X y no Y) | `AGENTS.md` (Decisiones arquitectonicas) |
| Algo que se descarta explicitamente construir | `AGENTS.md` (Fuera de alcance) |
| Cambio en el flujo de branches/PR/merge | `AGENTS.md` (Flujo de trabajo) + `.github/CONTRIBUTING.md` |
| Nuevo riesgo o cambio en el modelo de amenaza | `SECURITY.md` |

## Reglas criticas

- NUNCA inventar una decision que no esta documentada. Si no esta en
  `AGENTS.md`, preguntar antes de asumir.
- Un PR que cambia arquitectura pero no toca `AGENTS.md` se marca para
  revision — la doc no es opcional, es parte del cambio.
- Los comentarios en codigo explican el POR QUE local. `AGENTS.md` explica
  el POR QUE global (ver formato en Decision 1-6 del `PLAYBOOK.md` de
  `octalitycl/config` — cada decision documentada con problema real,
  alternativas descartadas, y solucion aplicada).
- Drift entre `AGENTS.md` y el codigo real: reconciliar en el mismo PR que
  lo causo, no dejarlo para despues.

## Anti-patrones

- Cambiar de proveedor/libreria en el codigo pero dejar `AGENTS.md`
  hablando del anterior.
- Documentar una decision solo en la descripcion del PR (se pierde cuando
  el PR se archiva) — copiarla a `AGENTS.md`.
