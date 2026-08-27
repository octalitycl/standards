---
name: testing-coverage
description: Use when writing a feature, fixing a bug, or reviewing a PR in an octalitycl Node/Vite repo. Enforces consistent test coverage — every feature and bug fix ships with tests.
license: MIT
---

# Testing Coverage — octalitycl (Node/Vite/Vitest)

## Stack

- Vitest (`npm test` -> `vitest run`)
- Coverage con `@vitest/coverage-v8`
- Un archivo de test por modulo: `<modulo>.test.ts`

## Regla de oro

- **Cada feature nueva requiere tests.** Sin excepciones, ni para la logica
  mas trivial (ver `NOTES.md` de este template para el piso de cobertura
  exigido en CI, no solo medido).
- **Cada bug fix requiere test de regresion** que FALLA sin el fix.
- Tests **deterministas**: sin `Math.random()`, sin dependencias de
  tiempo real, sin red real (mockear).
- Tests **rapidos**: si uno toma >2s, mockear la dependencia lenta.
- Sin tests que dependan del orden de ejecucion — cada uno aisla su estado.

## Estructura

```ts
// BIEN — describe comportamiento
it("returns 404 when user does not exist")

// MAL — describe implementacion
it("test getUser with invalid id")
```

## Que testear

1. **Black box** de logica de negocio: inputs/outputs esperados.
2. **Edge cases**: vacio, null, limites, caracteres especiales.
3. **Errores**: que pasa cuando falla, no solo el happy path.
4. **Contratos de API** si el repo tiene backend: status codes, schema de
   response, headers.
5. **Casos triviales tambien**: si `formatCLP(1000)` deberia devolver
   `"$1.000"`, hay un test que lo verifica explicitamente.

## Que NO testear

- Implementacion interna que no afecta el output observable.
- Codigo del framework (routing basico de Vite, lo que ya prueba la libreria).
- Tests de los propios mocks/fixtures.

## Ejemplo de test de regresion

```ts
// bug: formatCLP redondeaba mal con decimales exactos en .5
it("regression: rounds .5 up, not to even", () => {
  expect(formatCLP(1000.5)).toBe("$1.001");
});
```

## Ejecutar

```bash
npm test                          # todo
npm test -- --coverage            # con cobertura
npx vitest run <archivo>          # un archivo especifico
npx vitest run -t "nombre test"   # filtrar por nombre
```

## Anti-patrones

```ts
// MAL — depende del orden de ejecucion
let cache: number[] = [];
it("test a", () => { cache.push(1); });
it("test b", () => { expect(cache.length).toBe(1); }); // fragil

// MAL — solo happy path
it("processes input", () => { process("ok"); }); // sin assert de error

// BIEN — cubre error tambien
it("throws on empty input", () => { expect(() => process("")).toThrow(); });
```
