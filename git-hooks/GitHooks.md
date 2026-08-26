# Git Pre-commit Hook

## Que es y para que lo usamos
Un Git pre-commit hook es un script que Git ejecuta automaticamente antes de que se confirme cualquier `git commit`. 

La idea principal es no depender de la memoria de cada desarrollador para correr el linter o los tests:
- Si el codigo tiene errores de formato, los formatea solo y los mete en el commit.
- Si el linter encuentra violaciones de calidad, corta el commit y te avisa que corregir.
- Si algun test unitario falla, aborta el commit para que nadie pueda pushear codigo roto.

---

## Donde vive el script (`hooks/pre-commit`)

Por defecto, Git guarda los hooks adentro de `.git/hooks/`, pero esa carpeta **no es trackeada por Git** (nadie la comparte al clonar el repositorio).

Para solucionar esto, creamos la carpeta `hooks/` en la raiz del proyecto y adentro pusimos el script `hooks/pre-commit`:

```bash
#!/bin/sh
set -e

echo "Running pre-commit checks..."

# 1. Formatear
echo "Formatting code..."
./gradlew ktlintFormat detekt --auto-correct

# Re-agregar al stage cualquier archivo que el formatter haya corregido
git update-index --again

# 2. Verificar linters y analisis estatico
echo "Checking linters..."
./gradlew ktlintCheck detekt

# 3. Correr la suite de tests
echo "Running tests..."
./gradlew test

echo "All checks passed! Proceeding with commit."
```

### Detalles tecnicos importantes del script:
1. **`#!/bin/sh`**: El shebang en la linea 1 le dice al sistema que interprete el archivo como un shell script POSIX.
2. **`set -e`**: Hace que el script se corte inmediatamente en el primer comando que falle. Sin esto, si falla el linter pero el script llega al final, Git crearia el commit igual.
3. **`git update-index --again`**: Si el auto-formateo modifico algun archivo que ya habias preparado con `git add`, este comando actualiza el staging area para que el commit incluya las correcciones del formateador.

---

## Instalacion automatica con Gradle

Para que nadie tenga que copiar el archivo a mano a `.git/hooks/`, armamos una tarea de Gradle llamada `installGitHooks` adentro del convention plugin (`buildSrc`):

```kotlin
val installGitHooks = rootProject.tasks.maybeCreate<Copy>("installGitHooks").apply {
    description = "Copies git hooks from /hooks to /.git/hooks with execution permissions"
    group = "git hooks"
    from("${rootProject.rootDir}/hooks")
    into("${rootProject.rootDir}/.git/hooks")
    filePermissions {
        user {
            read = true
            write = true
            execute = true
        }
        group {
            read = true
            execute = true
        }
        other {
            read = true
            execute = true
        }
    }
}

tasks.matching { it.name in listOf("compileKotlin", "check", "test") }.configureEach {
    dependsOn(installGitHooks)
}
```

### Como funciona esta magia:
- La tarea usa `filePermissions` para asegurarse de que el archivo copiado a `.git/hooks/pre-commit` tenga permisos de ejecucion (`chmod +x` / `rwxr-xr-x`).
- Al estar conectada con `dependsOn` a `compileKotlin`, `test` y `check`, **se ejecuta sola** en segundo plano apenas compilas o corres un test en IntelliJ o en la consola.
- De esta forma, apenas alguien clona el repo y corre un test, ya tiene el hook instalado sin hacer nada.
