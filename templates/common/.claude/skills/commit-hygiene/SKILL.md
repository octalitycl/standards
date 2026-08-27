---
name: commit-hygiene
description: Use when creating, reviewing, or amending commits in an octalitycl repo. Enforces Conventional Commits format and a clean, reviewable history.
license: MIT
---

# Commit Hygiene — octalitycl

## Formato

```
type(scope): descripcion
```

```
^(feat|fix|chore|docs|style|refactor|perf|test|build|ci)(\(.+\))?!?: .+
```

Mismo regex que valida `guard.yml` en cada PR — si el commit no calza aca,
tampoco va a pasar CI.

## Types validos

| Type | Uso |
|---|---|
| feat | Nueva feature o comportamiento |
| fix | Bug fix |
| docs | Solo documentacion |
| chore | Mantenimiento, deps, tooling |
| refactor | Reestructuracion sin cambio de comportamiento |
| test | Agregar o arreglar tests |
| ci | Cambios en GitHub Actions |
| style | Formato, whitespace, sin logica |
| perf | Mejora de performance |
| build | Sistema de build |

## Scopes

<!-- Cada proyecto define su propia tabla de scopes segun su estructura real
     (ej: backend, frontend, api, infra, docs, ci). No hay un set universal —
     completar en el AGENTS.md del proyecto, no aca. -->

## Reglas criticas

- NUNCA `Co-Authored-By` ni atribucion de IA — `guard.yml` lo bloquea en el
  PR, y el hook local (si el proyecto tiene Husky+commitlint) lo bloquea
  antes.
- Un cambio logico por commit. No "add X and fix Y and update docs".
- Modo imperativo: "agrega" no "agrego", "corrige" no "corrigio".
- Subject <= 50 caracteres.
- NUNCA `--no-verify` para saltarse un hook.

## Ejemplos

```
feat(api): agrega endpoint de precios
fix(auth): valida HMAC antes de procesar webhook
refactor(cliente): extrae llamada http a servicio
ci: agrega chequeo de secret-scan
docs(arquitectura): agrega seccion de decisiones
chore: actualiza dependencias
```

## Anti-patrones

```
# MAL — vago
update stuff

# MAL — atribucion de IA
feat: add feature

Co-Authored-By: Claude <noreply@anthropic.com>

# MAL — multiples concerns en un commit
feat(api): add endpoint and fix webhook and update readme
```
