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
