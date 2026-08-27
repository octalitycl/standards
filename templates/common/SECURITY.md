# Security Policy — {{OWNER}}/{{REPO_NAME}}

## Alcance

<!-- Describir si hay backend, base de datos, y donde vive el dato sensible.
     Si es un sitio estatico sin backend, decirlo explicito: los datos viven
     en localStorage del navegador, no hay servidor que los reciba. -->

## Reportar una vulnerabilidad

Si encontras un problema de seguridad (por ejemplo, una credencial real
commiteada por error, o una vulnerabilidad de dependencias), reportalo en
privado a `{{OWNER}}` en vez de abrir un issue publico. No publiques
detalles del hallazgo hasta que este resuelto.

## Buenas practicas del repo

- Nunca commitear credenciales, tokens o API keys reales, ni siquiera de
  ejemplo. Usar placeholders (`<tu-api-key>`).
- `secret-scan` (gitleaks) corre en cada PR y push a `main`/`dev` para
  detectar secretos commiteados por error.
- Si el repo tiene dependencias (`package.json`, `requirements.txt`, etc.),
  Dependabot esta configurado para auditarlas — ver `.github/dependabot.yml`.
- Repo {{VISIBILIDAD}}; acceso restringido a: {{TEAM}}.
