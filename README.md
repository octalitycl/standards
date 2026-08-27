# config

Configuracion base de octalitycl para repos nuevos: convenciones de git,
plantillas de `.github/`, y el flujo de dos ramas (`main`/`dev`) que usamos
para sitios desplegados en Vercel con plan Hobby.

Nace de la configuracion real aplicada en tres repos privados del equipo (uno
React+Vite, dos sitios estaticos sin build) — sin nombrarlos aca porque son
privados. Este repo es la version generica y reutilizable de eso — la base
para el proximo repo de la empresa, no un archivo mas de documentacion suelta.

## Que hay aca

| Archivo | Para que |
|---|---|
| [`PLAYBOOK.md`](./PLAYBOOK.md) | Que se hizo, por que, y que limitaciones reales de plan (GitHub Free, Vercel Hobby) obligaron cada decision. Historial de los problemas encontrados y como se resolvieron. |
| [`CHECKLIST.md`](./CHECKLIST.md) | Pasos concretos, con comandos, para dejar un repo nuevo configurado igual que los anteriores. |
| [`templates/common/`](./templates/common/) | Archivos que van igual en CUALQUIER repo de la empresa (CODEOWNERS, CONTRIBUTING, issue templates, auto-assign, labeler, dependabot base, SECURITY.md, y 4 skills de Claude Code agnosticas al stack en `.claude/skills/`). |
| [`templates/static-site/`](./templates/static-site/) | Para sitios HTML/CSS/JS sin build: `AGENTS.md`, `guard.yml` sin depender de Node, `vercel.json`. |
| [`templates/node-vite-app/`](./templates/node-vite-app/) | Para apps con `package.json`/build: ESLint, Prettier, Husky+commitlint, Vitest, `guard.yml`/`ci.yml` con npm. |

## Cuando usar cada template

- **`static-site`**: el repo nuevo no tiene (ni va a tener) `package.json`.
  Es HTML/CSS/JS vanilla, con o sin algun script auxiliar en stdlib.
- **`node-vite-app`**: el repo nuevo tiene `package.json` real (React, Vite,
  o cualquier stack con build y tests posibles).

No copies ESLint/Husky/Vitest a un repo sin `package.json` solo por
"paridad" — es tooling sin nada que lintear/testear, y no se agrega
infraestructura que el proyecto no necesita.

## Que NO asumir

Cada repo nuevo puede diferir en algo real (ver `PLAYBOOK.md` seccion
"Cosas que hay que verificar, no asumir" antes de copiar):

- si tiene o no deploy en Vercel (define si hace falta `dev` + `vercel.json`)
- si el repo va a ser privado o publico (define si branch protection nativa
  de GitHub esta disponible gratis o no)
- quienes son los colaboradores reales de ESE repo (`gh api
  repos/<owner>/<repo>/collaborators`, no asumir que son los mismos 4 de
  siempre)
