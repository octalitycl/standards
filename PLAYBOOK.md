# Playbook — como llegamos a esta configuracion

Registro de decisiones para los repos de octalitycl. No es teoria: cada
punto de aca se probo en la practica en tres repos privados del equipo — acá
identificados solo por su stack, nunca por nombre — y cada "por que" tiene
una evidencia real detras (un log de CI, una respuesta de la API de GitHub,
un header HTTP).

> Los nombres reales de los repos y de los clientes NO van en este
> documento porque este repo es publico. Cuando hagas referencia a un caso
> real en un PR o issue de este repo, usa una descripcion generica del
> stack ("un sitio estatico en Vercel", "una app React+Vite"), nunca el
> nombre del repo/cliente.

## El problema de fondo

Vercel (plan Hobby, gratis) y GitHub (plan Free, repos privados) tienen
limitaciones reales que no se resuelven con mas configuracion — se resuelven
con un flujo de trabajo que las esquiva. Este documento es esa esquiva,
documentada para no tener que redescubrirla en cada repo nuevo.

## Decision 1: rama `dev` + `vercel.json` con `deploymentEnabled`

**Problema real:** el team de Vercel Hobby no permite invitar members — solo
la cuenta dueña del team (`octalitycl`) es member. Vercel bloquea cualquier
deploy (Preview o Production) cuyo commit este autorado por alguien que no
sea member, con el mensaje exacto:

> "The deployment was blocked because the commit author did not have
> contributing access to the project on Vercel. The Hobby Plan does not
> support collaboration for private repositories."

**Alternativas descartadas:**
- Pagar Vercel Pro para invitar members — el equipo decidio no pagar.
- GitHub Actions + Vercel CLI + token para generar el preview nosotros
  mismos — se armo, funciono, pero agregaba un secret mas que rotar
  (`VERCEL_TOKEN`) y no evitaba el problema en produccion. Se revirtio.

**Solucion aplicada:**
1. Rama `dev` de integracion. Los PRs de feature/fix/etc van contra `dev`,
   no contra `main`/`master`.
2. `vercel.json` -> `git.deploymentEnabled` desactiva el deploy de Vercel
   para `dev` y para toda rama `feat/*`, `fix/*`, etc. Vercel SOLO deploya
   `main` (produccion). Esto elimina el ruido de "Deployment was blocked" en
   cada PR de un contribuidor sin acceso al team — ya no lo intenta.
3. El merge a `main` (el unico deploy real que queda) lo hace **siempre**
   `octalitycl`, usando "Create a merge commit".

## Decision 2: "Create a merge commit", nunca squash

**Problema real, probado empiricamente (no supuesto):** se penso que bastaba
con que `octalitycl` apretara el boton de squash-merge para que Vercel viera
el commit como autorado por `octalitycl`. Se probo con un PR real: el
merge quedo con `merged_by: octalitycl`, pero el commit final seguia con
`commit.author: <autor original de la rama>`. Vercel siguio bloqueando el
deploy.

**Verificacion:** se abrio un segundo PR de prueba y se comparo
squash-merge vs "Create a merge commit" con `gh api
repos/<owner>/<repo>/commits/<sha> --jq .commit.author.name` en ambos casos.
Solo el merge commit real reasigna la autoria a quien ejecuta el merge.

**Solucion aplicada:** `allow_squash_merge=false`, `allow_rebase_merge=false`
a nivel de repo (`gh api --method PATCH repos/<owner>/<repo>`). Unico metodo
de merge permitido: "Create a merge commit".

## Decision 3: `guard.yml` en vez de branch protection nativa

**Problema real:** GitHub Free en un repo **privado** no permite branch
protection clasica ni Rulesets. Ambos endpoints devuelven:

```
403 "Upgrade to GitHub Pro or make this repository public to enable this
feature."
```

Se confirmo en los tres repos privados del equipo, con la cuenta dueña
(`octalitycl`, `admin: true` confirmado) — no es un problema de permisos, es
el plan.

La UI de GitHub (`Settings > Rules > New ruleset`) deja armar la
configuracion igual, pero avisa explicito: **"Your rulesets won't be
enforced on this private repository until you move to GitHub Team."** Se
puede dejar armada para el dia que el equipo pague Team, pero hoy no hace
nada.

**Excepcion real:** si el repo es **publico** (como este mismo,
`octalitycl/config`), branch protection nativa SI funciona gratis. La
condicion es "privado", no "Free" a secas.

**Solucion aplicada:** `.github/workflows/guard.yml`, un workflow que:
- bloquea push directo a `main`/`dev` (job `no-direct-push`, revisa que el
  ultimo commit sea un merge commit de PR);
- valida en cada PR: nombre de rama, titulo Conventional Commits, formato de
  cada commit, largo minimo del cuerpo del PR, ausencia de `Co-authored-by`;
- **`ownership-audit`**: en el evento `pull_request` `closed` con
  `merged == true` y `base_ref == main`, chequea
  `github.event.pull_request.merged_by.login == octalitycl`.

**Limitacion honesta de `guard.yml`, no ocultarla:** ninguno de estos jobs
puede impedir a nivel de Git que otra cuenta con acceso de escritura apriete
el boton "Merge". Corren DESPUES del evento y lo marcan en rojo, no bloquean
el click. Es un control social con tropiezo tecnico. Documentarlo asi en el
`AGENTS.md` de cada repo evita que el equipo confunda "hay un check" con
"esta bloqueado de verdad".

## Decision 4: Dependabot tiene que apuntar a `dev`, no al default branch

**Bug real que se encontro en produccion (no en diseño):** `dependabot.yml`
sin `target-branch` explicito abre PRs contra la rama default del repo
(`main`), no contra `dev`. Ademas, el nombre de rama que usa Dependabot
(`dependabot/npm_and_yarn/paquete-1.2.3`) nunca calza con la convencion
`feat|fix|chore|.../slug` que valida `guard.yml` — asi que **todos** los
PRs de Dependabot salian con el check de nombre de rama en rojo, siempre,
sin importar si el bump era seguro.

**Solucion aplicada:**
```yaml
- package-ecosystem: npm # o github-actions, etc.
  directory: "/"
  target-branch: dev
  schedule: { interval: weekly }
```
Y en `guard.yml`, una excepcion explicita para ramas `dependabot/*` en la
validacion de nombre.

**Gotcha de bootstrap:** Dependabot lee `dependabot.yml` desde la rama
default del repo (`main`), no desde `dev`. Si agregas `target-branch: dev`
en una rama de trabajo y la mergeas solo a `dev`, Dependabot va a seguir
abriendo PRs contra `main` hasta que promuevas ese cambio a `main` tambien.

## Decision 5: `delete_branch_on_merge` borra `dev` si la usaste como head

**Bug real, encontrado en vivo configurando uno de los sitios estaticos:**
`delete_branch_on_merge=true` (que si queremos, para limpiar ramas de
feature viejas) borra la rama HEAD de cualquier PR mergeado — sin excepcion
para `dev`. Cuando se abre el PR de promocion `dev -> main` y se mergea,
GitHub borra `dev` automaticamente porque era el head de ese PR.

**Solucion aplicada:** no es una config, es un paso de proceso. Documentado
en el `AGENTS.md` de cada repo: inmediatamente despues de mergear una
promocion `dev -> main`, quien la mergeo recrea `dev` desde `main`:

```bash
git checkout -B dev origin/main
git push origin dev   # (o --force-with-lease si dev todavia existe;
                       #  push normal si ya la borro, "stale info" en ese caso
                       #  es el --force-with-lease comparando contra un ref
                       #  que ya no existe, no un conflicto real)
```

## Decision 6: adaptar segun el stack real, no copiar a ciegas

**Error que se estuvo por cometer:** al pasar la configuracion de un repo
React+Vite+npm a un sitio estatico HTML/CSS/JS sin build, la primera
version copiaba ESLint, Husky+commitlint y Vitest tal cual. Se freno antes
de aplicarlo: no hay nada que lintear, ni un `package.json` para que Husky
se instale.

**Solucion aplicada:** dos templates separados (`static-site/` y
`node-vite-app/` en este repo), y una regla simple: **antes de replicar,
confirmar el stack real del repo** (`gh api repos/<owner>/<repo>/contents/`
buscando `package.json`, `requirements.txt`, etc.), no asumir que es igual
al ultimo repo que se configuro.

Igual de importante: **antes de asumir que un ajuste no aplica, verificarlo**.
Se penso que uno de los sitios estaticos no necesitaba el flujo
`dev`/`vercel.json` por no tener `package.json` — error: SI corria en
Vercel (confirmado con `curl -I` mirando el header `x-vercel-id`), aunque el
sitio fuera estatico. El criterio correcto no es "tiene build", es "tiene
deploy".

## Decision 7: `AGENTS.md` tambien documenta decisiones y no-goals

**Encontrado auditando repos personales del equipo (no un bug, una mejora
real):** dos proyectos distintos, sin coordinarse entre si, llegaron
independientemente al mismo patron: una seccion numerada de "decisiones
arquitectonicas" con el *por que* (ej. "no AWS: costos, se opto por VPS"),
y una seccion explicita de "no-goals" (que NO se va a construir, ej. "no
Kubernetes, no app movil"). Ninguna de las dos vive en un ADR separado —
van directo en el `AGENTS.md`, mas liviano.

**Solucion aplicada:** ambos templates (`static-site/AGENTS.md` y
`node-vite-app/AGENTS.md`) tienen ahora esas dos secciones como comentarios
placeholder, entre `Stack` y `Flujo de trabajo`.

## Decision 8: coverage se exige en CI, no solo se mide

**Encontrado en dos auditorias independientes:** correr `vitest run` (o
`pytest`) sin un flag de umbral mide la cobertura pero no la exige — un PR
puede bajarla a 0% y el build sigue verde. El numero solo sirve si un
descenso real hace fallar CI.

**Solucion aplicada:** `templates/node-vite-app/NOTES.md` documenta el flag
de umbral (`--coverage.thresholds.lines=...` en Vitest, o
`--cov-fail-under=...` en pytest) como parte del script de test, no como
paso separado opcional.

## Decision 9: patrones para repos mas grandes — documentados, no construidos

**Criterio:** no se agrega infraestructura que ningun repo actual necesita
(regla de "no premature abstraction"), pero se documenta aca para no
redescubrirla cuando aparezca el caso real:

- **CI con gate que se auto-saltea si falta el manifest** (visto en un repo
  personal del equipo): cada job de CI chequea si existe su manifiesto
  (`package.json`, `pyproject.toml`, `docker-compose.yml`) antes de correr
  pasos reales; si falta, el job sale verde via `if`. Permite marcar el
  check como "required" en branch protection desde el dia 0, antes de que
  exista el tooling — util para un repo que va a crecer por partes.
- **Docker multi-stage con usuario no-root** (`base` -> `development`/
  `build`/`production`, `COPY --from=build --chown=...`): para el dia que
  algun backend propio se dockerice.
- **CI de monorepo con `defaults.run.working-directory` +
  `cache-dependency-path` por subcarpeta**: para un repo con frontend y
  backend separados sin un tool de monorepo (Turborepo/Nx) de por medio.
- **Deploy que verifica liveness real** (poll a `/health`/`/ready` con
  retry, no solo "el contenedor esta corriendo"): para cualquier repo que
  se auto-hostee (Docker/VPS) en vez de usar un deploy gestionado como
  Vercel — ahi "Vercel: success" no aplica, hace falta verificar el proceso
  real.
- **Convencion dry-run/execute en scripts de mantenimiento** (ej.
  `ops:cleanup-branches` vs `ops:cleanup-branches:execute`): mismo
  principio de "nunca destructivo sin preview" que ya aplicamos en este
  mismo playbook (ver Decision 5) — nombrar los scripts de forma que el
  dry-run sea el default y la ejecucion real requiera un flag explicito.
- **`docs/` como carpeta separada** de `README.md`/`AGENTS.md` para
  documentacion pesada (manual de instalacion, modelo de base de datos,
  estructura del proyecto) — solo tiene sentido en un proyecto grande, no
  en los sitios estaticos/Vite chicos que armamos hasta ahora.
- **CodeQL** (SAST nativo de GitHub, gratis): confirmado que, igual que
  branch protection nativa, requiere que el repo sea **publico** — no
  aplica a los repos privados del equipo hoy, pero si aplica a este mismo
  repo (`octalitycl/config`) si en algun momento se quiere activar.

## Decision 10: skills de Claude Code repo-locales, no solo globales

**Encontrado auditando repos personales del equipo:** las skills de Claude
Code pueden vivir en `~/.claude/skills/` (global, solo en la maquina de
quien las configuro) o en `.claude/skills/<nombre>/SKILL.md` dentro del
propio repo (viaja con el codigo, cualquiera que clone el repo la tiene,
sin instalar nada aparte). Para un equipo de mas de una persona, la version
repo-local es la que realmente garantiza consistencia — una skill global no
la tiene automaticamente quien clona el repo en otra maquina.

**Tambien encontrado:** un repo tenia las mismas skills duplicadas en
`.claude/skills/` y `.agents/skills/`, pero `.claude/skills/<nombre>` era
en realidad un **symlink** a `../../.agents/skills/<nombre>` — un solo
archivo real, leible por Claude Code (`.claude/`) y por otra herramienta de
agentes (`.agents/`) sin duplicar contenido. Util el dia que el equipo use
mas de una herramienta de agentes sobre el mismo repo; no se implementa
ahora porque solo usamos Claude Code.

**Solucion aplicada:** `templates/common/.claude/skills/` trae 4 skills
agnosticas al stack (`branch-pr`, `commit-hygiene`, `docs-alignment`,
`issue-creation`), adaptadas de una skill real de un proyecto personal del
equipo pero corregidas para no contradecir nuestras propias reglas — la
version original asumia squash-merge como metodo de merge, que es
exactamente lo que tenemos prohibido (ver Decision 2). `testing-coverage`
(Vitest) vive solo en `templates/node-vite-app/`, no en `static-site`, que
no tiene tests automatizados por diseño.

## Decision 11: Node 24 / checkout v6 — documentado, no migrado aun

**Encontrado verificando el SHA de `gitleaks-action@v3` (v3.0.0 -> `e0c47f4`):**
las notas oficiales de esa release piden migrar `gitleaks-action` a runtime
Node 24 y, junto con eso, actualizar `actions/checkout` a `@v6`. Hoy
`templates/*/guard.yml` y `templates/*/ci.yml` usan `actions/checkout@v4` y
`actions/setup-node@v4` (Node 20) y siguen funcionando sin cambios.

**Por que no se migra ahora:** no es un breaking funcional hoy; `checkout@v4`
sigue corriendo. Migrar a `checkout@v6` + Node 24 ahora agregaria churn a
todos los repos sin beneficio inmediato. Se deja documentado para no perder la
fecha limite.

**Fecha limite real:** GitHub retira Node 20 de los runners hosteados el
**16 de septiembre de 2026** — a partir de ahi, Actions que sigan en Node 20
dejaran de correr. Antes de esa fecha hay que migrar `checkout@v4` -> `@v6`
(y cualquier otra Action anclada a Node 20) en ambos templates.

**Solucion aplicada:** no hay cambio de codigo ahora; se agrega este registro
para que no se pierda hasta septiembre. El pin de `gitleaks-action` ya quedo
en `e0c47f4f8be36e29cdc102c57e68cb5cbf0e8d1e # v3.0.0` (SHA verificado, firma
GitHub verificada).

## Cosas que hay que verificar, no asumir, en cada repo nuevo

1. `gh api repos/<owner>/<repo>/collaborators --jq '.[].login'` — no asumir
   que el equipo es siempre el mismo (uno de los repos privados no tenia a
   un colaborador del equipo al principio, se agrego despues).
2. `gh api repos/<owner>/<repo> --jq '.private'` — si es publico, branch
   protection nativa SI esta disponible gratis; usarla en vez de (o ademas
   de) `guard.yml`.
3. `curl -sI https://<homepage-del-repo>` buscando `x-vercel-id` — confirma
   si hay deploy real en Vercel antes de decidir si hace falta `dev`.
4. `gh api repos/<owner>/<repo>/contents/` buscando `package.json`,
   `requirements.txt`, `pyproject.toml` — define que template usar.
5. `gh repo view <owner>/<repo> --json defaultBranchRef` — el nombre de la
   rama default no siempre es `main` (uno de los repos privados usa
   `master`).
