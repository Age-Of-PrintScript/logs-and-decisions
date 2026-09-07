# Historial de decisiones
## Interfaz
Parser y automata interno. Expone: 
```kotlin
interface Parser {
    fun parse(tokens: List<Token>): Program
}
```

## Automata
### Versión 2

<img width="637" height="461" alt="image" src="https://github.com/user-attachments/assets/3990ee0f-15e0-436f-9f51-bb47735d1a1a" />


### Versión 1
Dado el "lenguaje de TOKENS":

<img width="780" height="61" alt="image" src="https://github.com/user-attachments/assets/cfaf40fe-689c-4f0b-b8c2-aebfa88f71da" />

Por ahora esta este automata

<img width="815" height="595" alt="image" src="https://github.com/user-attachments/assets/dbddff43-4ee6-42a8-abde-b8b7a98c7062" />

En escencia:
- Trata de "entender" que se quiere hacer en cada sentencia de codigo
- Tres posibles caminos: Declaración, Asignación y Call a una funcion
- ⚠️ La asignación es una Expressión. Por lo que hay que considerar casos donde hay sumas divisiones, operaciones aritmeticas
  por lo que por ahora tiene sentido es que en el flujo de codigo. el "automata" llame a otro componente que recursivamente
  pueda crear el arbol de ejecución de esa operación (ya que el automata finito no lo permite y pasar a un automata de pila
  es mas lío)

## Refactor v1.0 / Expresiones con Precedencia y ASTs Válidos

### ¿Qué cambiamos?
Antes el Parser dependía de enums viejos y tenía métodos separados para términos y factores (`separateExpression`, `separateTerm`) que no escalaban bien si metíamos operadores de comparación (`==`, `>`, `<`).

### 1. Inyección de Operadores y Precedencia
Ahora le inyectamos al Parser los operadores (`Operators.SUM`, `Operators.MULTIPLY`, etc.) donde cada uno sabe su `symbol` y su `precedence`:
- En el parsing de expresiones usamos precedencia dinámica (Precedence Climbing).
- Si el operador que sigue tiene mayor precedencia, se evalúa primero. Así se resuelve `2 + 3 * 4` sin tener que crear métodos hardcodeados para cada nivel de la gramática.

### 2. ASTType y validAST
Al parser le inyectamos la lista de sentencias válidas (`v1_0validAST`):
- `DECLARATION` (para `let`)
- `ASSIGNMENT` (para `x = ...`)
- `EXPRESSION_STATEMENT` (para `println(...)`)

Cuando el parser arranca en el estado `Start`, chequea si esa sentencia está permitida en la versión actual. Si en 1.0 alguien intenta meter algo que no va (como un `const` o un `if`), el parser lo frena de entrada.

### 3. Expresiones y AST simplificados
- `Expression.Literal` ahora guarda el valor y su `PSType` directamente sin andar adivinando tipos.
- Se agregó `Expression.Call` para que en el futuro funciones como `readInput()` puedan vivir adentro de expresiones y asignaciones.
- En `AST` se unificó `ExpressionStatement` para que cualquier llamada suelta como sentencia se maneje de forma genérica.
  
