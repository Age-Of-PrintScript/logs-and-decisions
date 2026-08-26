# Organizacion del Build con `buildSrc` y Convention Plugins

## Por que cambiamos la configuracion vieja
Antes teniamos en el `build.gradle.kts` de la raiz un bloque gigante con:
```kotlin
subprojects {
    plugins.withId("org.jetbrains.kotlin.jvm") {
        apply(plugin = "org.jlleitschuh.gradle.ktlint")
        apply(plugin = "io.gitlab.arturbosch.detekt")
        // ...
    }
}
```

Esto traia varios problemas de diseño:
1. **Acoplamiento feo**: Cada submódulo dependia implicitamente de cosas magicas que pasaban en el archivo raiz.
2. **Perdida de autocompletado y tipado (type-safe accessors)**: En Kotlin DSL, cuando aplicas plugins dinamicamente adentro de `subprojects`, Gradle no genera los metodos con tipado seguro como `detektPlugins("...")` o `detekt { ... }`. Habia que usar sintaxis rara con Strings como `add("detektPlugins", ...)`.
3. **Poco claro**: Al abrir el `build.gradle.kts` de un modulo (como `lexer`), no veias que plugins usaba ni que configuraciones tenia aplicadas.

La solucion estandar y recomendada por Gradle es usar **Convention Plugins en `buildSrc`**.

---

## Como funciona `buildSrc`

`buildSrc` es un directorio especial en la raiz que Gradle compila antes de ejecutar cualquier otra tarea del proyecto. Todo lo que pongas adentro de `buildSrc` queda disponible como plugins y librerias para los `build.gradle.kts` de tus modulos.

La estructura quedo armada asi:

```text
printscript/
├── buildSrc/
│   ├── build.gradle.kts
│   └── src/
│       └── main/
│           └── kotlin/
│               └── printscript.common-conventions.gradle.kts
├── build.gradle.kts (Raiz, completamente limpio)
├── lexer/
│   └── build.gradle.kts
├── parser/
│   └── build.gradle.kts
└── ...
```

---

## 1. El archivo `buildSrc/build.gradle.kts`

Aca le decimos a Gradle que `buildSrc` va a generar plugins en Kotlin DSL y declaramos las dependencias que nuestro convention plugin necesita:

```kotlin
plugins {
    `kotlin-dsl`
}

repositories {
    mavenCentral()
    gradlePluginPortal()
}

dependencies {
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin:2.2.0")
    implementation("org.jlleitschuh.gradle.ktlint:org.jlleitschuh.gradle.ktlint.gradle.plugin:12.2.0")
    implementation("io.gitlab.arturbosch.detekt:detekt-gradle-plugin:1.23.8")
    implementation("io.gitlab.arturbosch.detekt:detekt-formatting:1.23.8")
}
```

---

## 2. El plugin `printscript.common-conventions.gradle.kts`

Cualquier archivo `.gradle.kts` adentro de `buildSrc/src/main/kotlin/` se transforma automaticamente en un plugin de Gradle cuyo id es el nombre del archivo (en este caso, `printscript.common-conventions`).

Aca encapsulamos toda la configuracion comun:
- **Plugins base**: `kotlin("jvm")`, `java-library`, `ktlint`, `detekt`, `jacoco`.
- **Repositorio**: `mavenCentral()`.
- **Dependencias comunes**: JUnit 5, launcher y `detektPlugins(detekt-formatting)`.
- **Configuracion de Detekt**: Apuntando a `config/detekt/detekt.yml` con `autoCorrect = true`.
- **Configuracion de KtLint**: Version `1.5.0` y exclusion de scripts `.kts`.
- **Configuracion de JaCoCo**: Minimo de cobertura del 80% (`0.80`) enganchado a la tarea `check`.
- **Instalacion de Hooks**: Tarea `installGitHooks` para copiar el pre-commit hook con permisos ejecutables.

Al aplicar los plugins de forma directa con `plugins { ... }`, los bloques de configuracion son 100% tipados y no requieren hacks.

---

## 3. Como se usa en los modulos

Ahora cada modulo tiene un `build.gradle.kts` super limpio y declarativo.

Ejemplo en `lexer/build.gradle.kts`:
```kotlin
plugins {
    id("printscript.common-conventions")
}

dependencies {
    api(project(":tokens"))
}
```

Y el `build.gradle.kts` de la raiz quedo vacio:
```kotlin
// Root project build file - all conventions are managed by buildSrc/src/main/kotlin/printscript.common-conventions.gradle.kts
```

---

## Como agregar un modulo nuevo con este setup
1. Creas la carpeta del modulo en la raiz.
2. Le creas su `build.gradle.kts` con solo:
   ```kotlin
   plugins {
       id("printscript.common-conventions")
   }
   ```
3. Agregas la carpeta `src/main/kotlin`.
4. Lo incluyes en `settings.gradle.kts` con `include("tu_modulo")`.
5. Listo. Hereda automaticamente Kotlin JVM, Detekt, KtLint, JaCoCo, JUnit 5 y los hooks.
