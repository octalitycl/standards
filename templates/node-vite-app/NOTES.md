# Notas de este template

Lo que hay aca esta **verificado en produccion** en el primer repo
React+Vite+npm del equipo (privado): `guard.yml`, `dependabot.yml`,
`commitlint.config.js`, `.husky/commit-msg`.

Lo que **falta a proposito**, porque depende de cada proyecto y copiarlo
ciego rompe cosas: `eslint.config.js`, `.prettierrc.json`, `vitest.config.ts`,
y los scripts de `package.json`. Armarlos especificos al framework de este
repo (React vs Vue vs vanilla TS, etc.), tomando como referencia de partida
un repo real del equipo con stack similar, no como copia literal.

## Scripts de `package.json` que `ci.yml` y `guard.yml` esperan

```json
{
  "scripts": {
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "build": "vite build",
    "prepare": "husky"
  }
}
```

Si el proyecto no tiene test runner todavia: configurarlo ANTES de escribir
codigo de produccion. No es opcional — ver `rules/common/testing.md` si el
equipo usa Claude Code, o el equivalente en las normas internas.

## Coverage: exigirlo en CI, no solo medirlo

Confirmado en dos auditorias independientes de repos reales del equipo: medir
cobertura sin que falle el build no sirve de piso minimo, solo de metrica
decorativa. Agregar el flag de umbral al comando de test, no solo dejar que
`vitest run` reporte el numero:

```json
"test": "vitest run --coverage --coverage.thresholds.lines=70 --coverage.thresholds.branches=60"
```

O equivalente en `vitest.config.ts` (`test.coverage.thresholds`). El numero
exacto lo define cada proyecto; lo que no es opcional es que un PR que baja
la cobertura por debajo del piso falle CI, no solo lo reporte.

## Dependencias devDependencies minimas para que `guard.yml`/hooks funcionen

```
eslint, @eslint/js, typescript-eslint, globals, eslint-config-prettier,
prettier, husky, @commitlint/cli, @commitlint/config-conventional,
vitest (+ @vitest/coverage-v8 si se mide cobertura)
```

## `vercel.json`

Este template no trae un `vercel.json` completo (cada proyecto tiene su
propio `buildCommand`/`outputDirectory`/headers). Fusionar el bloque de
`vercel.json.git-block-snippet.json` dentro del `vercel.json` que ya tenga
el proyecto — no reemplazarlo entero.

## `.husky/commit-msg`

Necesita permiso de ejecucion despues de copiarlo:

```bash
chmod +x .husky/commit-msg
```
