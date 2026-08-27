# Contribuir — {{REPO_NAME}}

Flujo obligatorio para el equipo ({{TEAM}}).

## 1. Crea tu rama desde `dev`

```bash
git checkout dev && git pull origin dev
git checkout -b feat/mi-feature   # o fix/, chore/, docs/, style/, refactor/, perf/, test/, build/, ci/
```

Formato: `^(feat|fix|chore|docs|style|refactor|perf|test|build|ci)/[a-z0-9._-]+$`
Enforced por `guard.yml`. La rama `dev` solo se permite como head cuando el PR base es `main` (promocion).

> Si este repo no tiene `dev` (no hay deploy que proteger), cambia `dev` por
> `main` en este documento.

## 2. Commits — Conventional Commits

```
type(scope): mensaje en imperativo
```

`type` en `feat|fix|chore|docs|style|refactor|perf|test|build|ci`. Sin `Co-authored-by` ni rastro de IA.

## 3. Abre PR contra `dev`

Usa `.github/PULL_REQUEST_TEMPLATE.md`. Descripcion >= 20 chars. CI debe estar verde.

## 4. CI verde + 1 approval

`guard.yml` valida rama, titulo, formato de commits y body. `ci.yml` corre
`secret-scan` (gitleaks) y, si el repo tiene `package.json`, lint/typecheck/test/build.
Necesitas al menos 1 approval antes de mergear.

## 5. Merge a `dev` — cualquiera del equipo

Con 1 approval y CI verde, **cualquiera** del equipo puede mergear a `dev`
usando **"Create a merge commit"** (NUNCA squash ni rebase). Si el repo
despliega en el team de Vercel Hobby, el merge lo tiene que hacer
`{{OWNER}}` (unico member) para que Vercel genere el Preview Deployment.

## 6. Promocion `dev` -> `main` — SOLO `{{OWNER}}`

Cuando `dev` esta listo para prod, se abre PR `dev` -> `main`. **Solo
`{{OWNER}}` (dueño) puede mergearlo**, con **"Create a merge commit"**
(NUNCA squash). El job `ownership-audit` de `guard.yml` audita despues del
merge que lo haya hecho `{{OWNER}}`.

Despues de la promocion, `{{OWNER}}` recrea `dev` desde `main` de inmediato
(`delete_branch_on_merge` la borra automaticamente al usarla como head del PR):

```bash
git checkout -B dev origin/main
git push origin dev
```

> Cuando el repo pase a GitHub Team/Pro (o si ya es publico), reemplazar
> este workaround por branch protection nativa. Ver `PLAYBOOK.md` en
> `octalitycl/config`.
