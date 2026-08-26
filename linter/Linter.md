# Linter (Detekt)

## Que es y para que lo usamos
Para el analisis estatico de codigo usamos Detekt. A diferencia de un formatter que solo mira como se ve el codigo (espacios, saltos de linea), el linter analiza la calidad interna: complejidad ciclomática, metodos gigantes, malas practicas de Kotlin, posibles bugs (null safety mal usada, cast inseguros) y code smells generales.

Toda la configuracion de las reglas vive en `config/detekt/detekt.yml`.

---

## Decisiones tomadas en la configuracion

Detekt por defecto viene con reglas muy estrictas pensadas para apps generales, pero para un interprete/compilador como PrintScript varias cosas chocaban de frente contra como se programa un parser o un evaluador de AST. Ajustamos las siguientes cosas:

### 1. Cantidad de returns y throws (ReturnCount y ThrowsCount)
- Por defecto Detekt te frena si metes mas de 2 returns o throws en una funcion.
- En un parser con maquinas de estados o en el `ExpressionSolver`, tener guard clauses o retornos tempranos para cada tipo de token/operador es super comun y natural.
- Subimos `ReturnCount` a 15 y `ThrowsCount` a 4, activamos `excludeGuardClauses: true` y excluimos los tests.

### 2. Complejidad en los when (CyclomaticComplexMethod)
- Cuando haces un `when (token.type)` o `when (node)` con 10 ramas simples, la complejidad ciclomática vuela por los aires si Detekt cuenta cada branch como un camino complejo.
- Activamos `ignoreSingleWhenExpression: true` e `ignoreSimpleWhenEntries: true` para que los `when` exhaustivos sobre tipos de tokens o AST no tiren falsos positivos.

### 3. Nombres de tokens y paquetes (ClassNaming y PackageNaming)
- En `tokens/Token.kt` tenemos objetos singleton como `object OPEN_PARENTHESIS : TokenType` escritos en mayusculas. `ClassNaming` por defecto exige PascalCase estricto, asi que cambiamos el patron a `[A-Z][_a-zA-Z0-9]*` para que acepte nombres de constantes.
- `PackageNaming` no permitia guiones bajos en los subpaquetes (como `parser.states.call_branch`). Cambiamos el regex a `^[a-z]+(\.[a-z][A-Za-z0-9_]*)*$` para soportarlo.

### 4. Permisividad durante el desarrollo
- **ForbiddenComment**: Por defecto rompe si dejas un `TODO:`. Lo desactivamos (`active: false`) para poder dejar placeholders mientras trabajamos en una feature sin que explote el build.
- **MagicNumber**: Excluimos los tests y agregamos a la lista de ignorados numeros comunes (`-1, 0, 1, 2, 3, 4, 5, 10, 100`).
- **WildcardImport**: Prohibido en codigo principal, pero con `excludes: ['**/test/**']` para poder importar `org.junit.jupiter.api.Assertions.*` comodamente en los tests.

---

## Como se corre

Para analizar todo el proyecto:
```bash
./gradlew detekt
```

Si hay violaciones de formato que Detekt pueda auto-corregir:
```bash
./gradlew detekt --auto-correct
```

Los reportes en HTML quedan generados en `<modulo>/build/reports/detekt/detekt.html`.
