# Formatter (KtLint y Detekt Formatting)

## Que es y para que lo usamos
El formateador se encarga exclusivamente del aspecto visual y estilo del codigo:
- Indentacion (4 espacios).
- Espaciado alrededor de llaves, dos puntos, operadores y palabras clave.
- Limite maximo de caracteres por linea (180 caracteres).
- Salto de linea al final de cada archivo.
- Eliminacion de espacios en blanco al final de linea (trailing spaces).
- Orden alfabetico de imports y eliminacion de imports sin uso.

En el proyecto conviven dos cosas que usan las mismas reglas de KtLint:
1. El plugin oficial `org.jlleitschuh.gradle.ktlint` aplicado a los modulos.
2. El modulo `detekt-formatting` adentro de Detekt.

---

## Configuracion (.editorconfig)

KtLint no usa archivos YAML ni Kotlin para configurar sus reglas, usa el estandar `.editorconfig` en la raiz del repo. Ahi definimos:

```editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true
max_line_length = 180

[*.{kt,kts}]
ktlint_standard_package-name = disabled
ktlint_standard_filename = disabled
ktlint_standard_class-naming = disabled
ktlint_standard_no-wildcard-imports = disabled
ktlint_standard_argument-list-wrapping = disabled
ktlint_standard_property-naming = disabled
```

### Por que desactivamos esas reglas puntuales:
- `package-name`: Para que no se queje de los paquetes con guiones bajos (`call_branch`, `assignment_branch`).
- `filename`: Para que acepte archivos utilitarios que no se llaman igual a la clase interna (como `utils.kt` o `Error.kt` que tiene `LexerError`).
- `class-naming`: Para que acepte los objetos de tokens en mayusculas (`OPEN_PARENTHESIS`).
- `no-wildcard-imports`: Para poder meter comodines en tests.

---

## Problemas que tuvimos y como los resolvimos

### 1. Error de parseo en los archivos build.gradle.kts
El plugin de Gradle de KtLint por defecto intenta formatear dos cosas:
- Los archivos `.kt` de `src/main/kotlin`.
- Los scripts `.kts` como `build.gradle.kts` (mediante las tareas `runKtlintFormatOverKotlinScripts`).

El parser de KtLint no entiende la sintaxis de Gradle Kotlin DSL y tiraba `KtLint failed to parse file: .../build.gradle.kts`. Lo solucionamos en el convention plugin desactivando esas tareas y excluyendo los `.kts`:
```kotlin
configure<org.jlleitschuh.gradle.ktlint.KtlintExtension> {
    filter {
        exclude("*.kts")
        exclude("**/*.kts")
    }
}

tasks.matching { it.name.contains("KotlinScript") }.configureEach {
    enabled = false
}
```

### 2. Soporte para guards en when (Kotlin 2.1)
En `ExpressionSolver.kt` usamos guard conditions dentro de los `when` (`is NumberLiteral if right is NumberLiteral ->`).
La version vieja de KtLint que venia por defecto no conocia esta sintaxis nueva de Kotlin 2.1 y tiraba `Expecting '->'`. Lo resolvimos fijando la version de KtLint a `1.5.0` en el convention plugin:
```kotlin
configure<org.jlleitschuh.gradle.ktlint.KtlintExtension> {
    version.set("1.5.0")
}
```

---

## Como se corre

Para formatear automaticamente todo el codigo que este desalineado:
```bash
./gradlew ktlintFormat
```

Para solo chequear si hay errores de formato sin tocar los archivos:
```bash
./gradlew ktlintCheck
```
*(Nota: si hay errores, ktlintCheck corta la ejecucion de Gradle con fallo para avisarte que falta formatear).*
