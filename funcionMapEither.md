## Either Extension Method
### Método
```
inline fun <L, R> Either<L, R>.getOrReturn(onFailure: (L) -> Nothing): R {
    return when (this) {
        is Failure -> onFailure(this.value)
        is Success -> this.value
    }
}
```
Esta funcion tiene varias cosas extrañas:
- Tiene la palabra **inline** adelante
- Su return type es **R** (o Success) aunque la rama de Failure retorna el Failure
  
    ```
    state.consume(chr).getOrReturn { return Failure(it) }
    ```

- La funcion onFailure retorna **Nothing**

### Explicación
La idea de esta función es poder reemplazar los 15 when anidados en el LexerStateMachine:
```
when (result) {
  is Failure -> return Failure(result.value)
  is Success -> {
      when(val appendResult = builder.addChar(chr)){
          is Failure -> return Failure(appendResult.value)
          is Success -> {
              state = result.value

              val isLastChar = i == source.length - 1
              val nextChar = if (!isLastChar) source[i + 1] else null

              val shouldCloseToken = nextChar == null || !state.canConsume(nextChar)

              if (shouldCloseToken) {
                  when(val buildResult= builder.build()){
                      is Failure -> return Failure(buildResult.value)
                      is Success -> {
                          tokenList.add(buildResult.value)
                          builder.reset()
                          state = InitialState()
                      }
                  }
              }
          }
      }
  }
          
```
Basicamente, cada vez que llego a la rama Failure quiero cortar la funcion tokenize (dentro del cual está ese código). Eso se repite para los 3 when. De la misma forma, si es un Success quiero simplemente usar ese valor. Para eliminar esta repetición, sirve el getOrReturn.


Entonces, lo que hace la funcion es: 
- Si es un Success -> devuelve el valor del Success
- Si el resultado es un Failure -> corta tokenize (no solo getOrReturn)

  Como hace esto?
  
  Si el resultado es un Failure, se ejecuta la funcion lambda onFailure:
      ```
      { return Failure(it) }
      ```
  Esta función recibe el valor de error y retorna un **Nothing**. Este Nothing lo que hace es avisarle al compilador que nunca va a devolver un valor "normal", siempre va a terminar con un return (osea el return solo, que corta la funcion donde está escrito ese return) o con una excepción. Así, evita el problema con el return type **R** de la función.

  Ahora, para cortar tokenize (no solo getOrReturn), necesita la keyword **inline**. Esta palabra lo que hace es que, al momento de compilar la función, "copia" el codigo que está adentro del getOrReturn en donde es llamada. Eso lo que implica es que el codigo se "transforma" en:
  ```
  val temp = state.consume(chr)
    when (temp) {
        is Failure -> return Failure(temp.value) 
        is Success -> temp.value
    }
  ```
    Entonces el return de la lamda es como si estuviera directamente adentro de tokenize, y por ende corta la funcion.
