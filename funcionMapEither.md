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
Basicamente, cada vez que llego a la rama Failure quiero cortar el for dentro del cual está ese codigo. Eso se repite para los 3 when. De la misma forma, 
si es un Success quiero simplemente usar ese valor. Para eliminar esta repeticion, sirve el getOrReturn.
