# Interpreter
el interpreter expone esta interfaz
``` kotlin
interface Interpreter {
    fun execute(program: Program): Either<RuntimeError, ExecutionResult>
    fun executeWithEnvironment(program: Program, runtimeEnvironment: RuntimeEnvironment): Either<RuntimeError, ExecutionResult>
}
```

## Que siginfica el return Execution Result?
El return es un either entre runtime error (ya que al estar ejecutando ya los errores son de runtime) o un Execution result
Este ExecutionResult es un dto con:
- RunTimeEvents: Todos los side-effects y cosas que pasaron durante la ejecucción se guardan acá. Ej: los prints. En vez de directamente
llamar al println de kotlin. guardamos lo que sería la salida del programa en ese DTO. Así. cuando alguien consuma la api de nuestro sistema
simplemente recibe los eventos y elige la forma correcta de mostrar los resultados

- RunTimeEnvironment: Toda la memoria de la ejecucción se guarda ahí. Si queremos ver en que estado quedó o queremos seguir ejecutando
sobre el mismo environment simplemente retroalimentamos el sistema con el mismo environment

## Como funciona internamente
muy simple, toma cada caso del ast (Declaration, Assignment o Call) y lo resuelve. No hay mucho que explicar
- Si es una declaration chequea que la variable no exista en el env. Si existe pincha, sino la crea con el valor dado
- Si es assignment es al reves, chequea que exista y le cambia el valor
- si es un call. guarda el evento generado (por el println) en runTimeEvents

## Refactor v1.0 / Inyección de Operaciones y Built-in Functions

### ¿Qué cambiamos?
Antes el Interpreter tenía un `when` gigante con cada caso posible de suma, resta, tipos cruzados y `PrintScriptFunctions.PRINTLN` todo metido en el medio del código. Si agregábamos un tipo nuevo o una función nueva, había que modificar todo el Interpreter.

### 1. Inyección de Built-in Functions
Ahora las funciones son contratos (`Function` con lambda):
- Le inyectamos `v1_0builtInFunctions` (un mapa `Map<String, Function>`).
- Para `println`: toma el argumento, crea un `PrintEvent(mensaje)` y lo devuelve en `FunctionResult`.
- El Interpreter no sabe qué hace cada función, solo busca el nombre en el mapa, la ejecuta y junta los eventos producidos en `RuntimeEvents`.

### 2. Inyección de Operaciones Binarias (`binaryOperationMap`)
En vez de tener `when (left is Number && right is Number)` en el medio del evaluador:
- Le inyectamos `v1_0binaryOperations` (un mapa de `OperationKey(operador, tipoIzq, tipoDer)` a su función `BinaryOperation`).
- Ejemplo: `OperationKey(SUM, NumType, NumType)` ejecuta la suma matemática.
- Ejemplo: `OperationKey(MULTIPLY, NumType, StrType)` ejecuta la repetición/concatenación del string si el número es entero.

Cuando el Interpreter evalúa una operación (`Expression.Operation`), simplemente busca en la tabla `(operador, izq.type, der.type)` y ejecuta la lambda correspondiente. Cero `when`s de tipos hardcodeados en el núcleo del intérprete.
