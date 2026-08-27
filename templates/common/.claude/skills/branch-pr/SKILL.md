---
name: branch-pr
description: Use when creating a branch, opening a pull request, or preparing changes for review in an octalitycl repo. Standardizes branch naming, PR title/body format, and merge method.
license: MIT
---

# Branch & PR — octalitycl

## Nombre de rama

```
^(feat|fix|chore|docs|style|refactor|perf|test|build|ci)\/[a-z0-9._-]+$
```

| Type | Ejemplo |
|---|---|
| feat | `feat/endpoint-precios` |
| fix | `fix/validacion-webhook` |
| docs | `docs/actualizar-readme` |
| chore | `chore/actualizar-deps` |
| refactor | `refactor/servicio-x` |
| test | `test/regresion-cron` |
| ci | `ci/agregar-check` |
| perf | `perf/cache-consultas` |

Descripcion en minusculas, solo `a-z`, `0-9`, `.`, `_`, `-`.

## Titulo de PR (Conventional Commits)

```
^(feat|fix|chore|docs|style|refactor|perf|test|build|ci)(\(.+\))?!?: .+
```

Ejemplos: `feat(api): agrega endpoint de precios`, `fix(auth): valida HMAC antes de procesar webhook`.

## Flujo

1. Si el repo usa el modelo de dos ramas: `git checkout dev && git pull && git checkout -b type/descripcion`. Si solo tiene `main`, ramificar desde ahi.
2. Commits atomicos — un cambio logico por commit (ver skill `commit-hygiene`).
3. Validar local antes de pushear (lint/typecheck/test/build segun el proyecto).
4. `git push -u origin type/descripcion`.
5. `gh pr create` — el template de `.github/PULL_REQUEST_TEMPLATE.md` se carga solo.
6. Descripcion del PR minima 20 caracteres, formato "Que cambia / Por que / Como se probo" — `guard.yml` lo valida.

## Merge: "Create a merge commit", NUNCA squash

**Distinto de otros proyectos personales del equipo, a proposito.** Los
repos de octalitycl con deploy en Vercel Hobby necesitan que el commit final
quede autorado por quien mergea (ver `PLAYBOOK.md` de `octalitycl/config`,
Decision 2). Squash-merge conserva el autor original de la rama y rompe el
deploy. Por eso:

- Squash y rebase estan deshabilitados a nivel de repo.
- El unico metodo de merge es "Create a merge commit".
- A `dev` (si existe): cualquiera del equipo. A `main`: solo el owner
  (`octalitycl`).

## Reglas criticas

- NUNCA `git push --force`/`--force-with-lease` sobre una rama compartida.
- NUNCA commits de WIP, `auto-save:`, o basura — usar `git reset -p` o
  `git rebase -i` ANTES del primer push, nunca despues.
- NUNCA `git commit --amend` de un commit ya pusheado a una rama compartida.
- NUNCA `Co-Authored-By` ni rastro de autoria de IA.
- NUNCA `--no-verify` para saltarse un hook — si bloquea, se arregla la
  causa, no el sintoma.
