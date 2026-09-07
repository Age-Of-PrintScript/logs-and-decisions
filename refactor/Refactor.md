# Gran Refactor de PrintScript: Desacoplar Domain y Versiones Dinámicas

## ¿Qué problema teníamos?
Teníamos todo recontra acoplado al módulo `domain`. En `domain/Rules.kt` estaban definidos enums fijos para todo: `PrintScriptType`, `PrintScriptFunctions`, `PrintScriptOperator` y `PrintScriptSymbols`.

El problema con esto es que no era nada extensible:
- Si queríamos soportar una nueva versión del lenguaje (como la 1.1) con nuevos tipos (`boolean`), nuevas palabras clave (`const`, `if`) o nuevas funciones (`readInput`), teníamos que modificar directamente los enums del domain.
- Todos los módulos (`lexer`, `parser`, `interpreter`) hacían `when` gigantes y pattern matching contra esos enums cerrados.
- En tiempo de ejecución no había forma de cambiar de versión sin cambiar el código base.

Básicamente, el lenguaje no podía crecer sin romper todo a su paso.

---

## ¿Cuál fue la solución?
La idea central fue **sacar casi todo lo hardcodeado de `domain`** y convertirlo en interfaces abiertas y genéricas:
- `PSType`: para representar cualquier tipo de dato (con `StrType` y `NumType` como básicos).
- `PSLiteral`: guarda el texto crudo (`raw`) y su `PSType`.
- `PSOperator`: operador abierto que define su símbolo (`symbol`) y su nivel de precedencia (`precedence`).
- `PSFunction`: descriptor genérico de funciones.

Ahora, el que tiene el control de las versiones es el **`engine`** (por ejemplo en el paquete `engine/ps_versions/v1_0/`), y es quien le **inyecta** a cada módulo exactamente lo que necesita para funcionar:

1. **Al Lexer**: le inyecta el mapa de keywords (`v1_0keywords`) y el mapa de símbolos (`v1_0Symbols`).
2. **Al Parser**: le inyecta la lista de operadores con sus precedencias (`Operators`) y la lista de sentencias válidas (`v1_0validAST`).
3. **Al Interpreter**: le inyecta la tabla de operaciones binarias (`v1_0binaryOperations`) y el mapa de funciones incorporadas (`v1_0builtInFunctions`).

---

## ¿Cómo conviven los cambios para no romper todo?
Para poder trabajar tranquilos y que el proyecto siga compilando y testeando en verde en cada commit, usamos la estrategia de cambio en paralelo (*Parallel Change*):
1. Renombramos las cosas viejas a `TokenViejo`, `ASTViejo`, `ExpressionViejo`, etc.
2. Creamos en paralelo los nuevos `Token.kt`, `ASTs.kt`, `Expression.kt` limpios y desacoplados.
3. Definimos todas las dependencias e inyecciones de `v1_0` en el Engine.
4. Ahora cada integrante del equipo puede refactorizar su módulo (`lexer`, `parser`, `interpreter`) en su propia branch sin pisarse con los demás.
5. Cuando todo esté funcionando con lo nuevo, conectamos todo en el `Engine` y borramos lo viejo definitivamente.
