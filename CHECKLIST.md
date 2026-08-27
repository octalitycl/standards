# Checklist — dejar un repo nuevo configurado

Sustituye `<owner>`, `<repo>`, `<homepage>` y la lista de colaboradores por
los valores reales. No copies estos comandos sin adaptarlos — cada paso
tiene un "verificar" antes del "aplicar" a proposito (ver `PLAYBOOK.md`
seccion final).

## 0. Verificar antes de tocar nada

```bash
gh repo view <owner>/<repo> --json defaultBranchRef,isPrivate,description,homepageUrl
gh api repos/<owner>/<repo>/collaborators --jq '.[].login'
gh api repos/<owner>/<repo>/contents/ --jq '.[].name'
curl -sI https://<homepage> | grep -i x-vercel-id   # confirma deploy en Vercel, si aplica
```

Con eso decidis: `static-site` o `node-vite-app` (`README.md` de este repo),
y si hace falta el flujo `dev` (solo si hay deploy real que proteger).

## 1. Settings de repo (via API, no a mano en la UI)

```bash
gh api --method PATCH repos/<owner>/<repo> \
  -f allow_squash_merge=false \
  -f allow_rebase_merge=false \
  -f allow_merge_commit=true \
  -F delete_branch_on_merge=true \
  -F has_discussions=true
```

Si el repo es **publico**, ademas evaluar branch protection nativa en vez
de (o ademas de) `guard.yml` — esta disponible gratis:

```bash
gh api repos/<owner>/<repo>/branches/main/protection 2>&1   # sin 403 = disponible
```

## 2. Clonar y crear `dev` (solo si hay deploy en Vercel que proteger)

```bash
gh repo clone <owner>/<repo> ~/Developer/<repo>
cd ~/Developer/<repo>
git checkout -B dev origin/main   # o origin/master, segun defaultBranchRef
git push -u origin dev
git checkout -b chore/team-repo-setup
```

Si NO hay deploy que proteger, saltar la rama `dev`: trabajar directo contra
`main` con el mismo modelo de PR + `octalitycl` como unico merger.

## 3. Copiar templates

```bash
cp -r templates/common/.github ~/Developer/<repo>/.github
cp templates/common/SECURITY.md ~/Developer/<repo>/SECURITY.md

# segun el stack (ver README.md de este repo):
cp -r templates/static-site/. ~/Developer/<repo>/      # o
cp -r templates/node-vite-app/. ~/Developer/<repo>/
```

Reemplazar en todos los archivos copiados:
- `{{REPO_NAME}}` -> nombre real del repo
- `{{OWNER}}` -> `octalitycl` (o el owner real)
- `{{TEAM}}` -> lista real de colaboradores (paso 0)
- `{{HOMEPAGE}}` -> dominio real, si aplica

```bash
grep -rl '{{' ~/Developer/<repo>/.github ~/Developer/<repo>/AGENTS.md 2>/dev/null
# reemplazar con sed o a mano, uno por uno — no hacer un sed -i global
# ciego sobre todo el repo, puede tocar archivos que no son templates
```

## 4. Primer PR: setup completo

```bash
git add -A
git commit -m "chore(repo): configure tooling and conventions for team collaboration"
git push -u origin chore/team-repo-setup
gh pr create --base dev --head chore/team-repo-setup \
  --title "chore(repo): configure tooling and conventions for team collaboration" \
  --body "Que cambia / Por que / Como se probo — minimo 20 caracteres, guard.yml lo exige."
```

Esperar CI en verde (`gh pr checks <numero>`) antes de mergear.

```bash
gh pr merge <numero> --merge
```

## 5. Promover a `main` (solo si se uso `dev`)

```bash
git fetch origin dev main
gh pr create --base main --head dev --title "chore(repo): promote dev to main" --body "..."
gh pr checks <numero>
gh pr merge <numero> --merge
```

**Inmediatamente despues**, recrear `dev` (la borro `delete_branch_on_merge`):

```bash
git fetch origin main
git checkout -B dev origin/main
git push origin dev   # o --force-with-lease si el fetch de dev no esta stale
```

## 6. Verificar, no asumir

```bash
gh api repos/<owner>/<repo>/commits/<sha-de-la-promocion>/status --jq '.statuses[] | {context, state}'
# esperar "Vercel" / "success" si aplica

diff <(git rev-parse origin/main) <(git rev-parse origin/dev) && echo IGUALES
```
