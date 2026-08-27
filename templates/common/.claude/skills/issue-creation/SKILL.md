---
name: issue-creation
description: Use when creating, drafting, or triaging a GitHub issue in an octalitycl repo. Standardizes issue-first workflow so PRs always link to a clear, agreed-upon objective.
license: MIT
---

# Issue Creation — octalitycl

## Alcance real de este skill

**No es CI-enforced hoy.** A diferencia de otros proyectos donde un workflow
bloquea el merge si el PR no linkea un issue aprobado, `guard.yml` en los
repos de octalitycl actuales NO valida esto — es disciplina de equipo, no
un gate tecnico. Si se quiere agregar ese gate, ver `PLAYBOOK.md` de
`octalitycl/config`, Decision 9 (esta documentado como patron para repos
mas grandes, no construido todavia).

## Flujo (issue-first, recomendado)

1. Identificar tipo: Bug o Feature.
2. Crear con plantilla (`gh issue create --template bug_report.md` o
   `feature_request.md` — ya vienen en `.github/ISSUE_TEMPLATE/`). Issues en
   blanco estan deshabilitados (`blank_issues_enabled: false`).
3. Completar todos los campos de la plantilla.
4. Si el proyecto usa labels de aprobacion (`status:approved`), esperarla
   antes de abrir el PR correspondiente.

## Campos que debería tener un buen issue

**Bug:**
- Pasos exactos para reproducir
- Comportamiento esperado vs actual
- Entorno (local / preview / prod)
- Evidencia (logs, screenshots)

**Feature:**
- Problema u oportunidad ANTES que la solucion propuesta
- Alternativas consideradas y por que no
- Criterios de aceptacion verificables (checkboxes)

## Reglas criticas

- SIEMPRE usar plantilla, nunca issue en blanco.
- Un issue describe UN objetivo. Si crece, dividir en varios.
- Feature sin problema/motivacion explicito: pedir que lo agreguen antes de
  triarlo.
- Bug sin pasos de reproduccion: pedir repro antes de triarlo.
- **Seguridad (secrets, auth, fuga de datos): NUNCA issue publico.**
  Reportar en privado segun `SECURITY.md` del repo, nunca en un issue
  visible para cualquiera con acceso de lectura.

## Anti-patrones

- Abrir un PR sin ningun issue que explique el objetivo — funciona para un
  fix trivial de una linea, no para una feature.
- Issue de feature que ya viene con la solucion decidida sin mencionar el
  problema — dificulta que otro proponga una alternativa mejor.
