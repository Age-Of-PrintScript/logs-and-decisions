# Historial de decisiones
## Interfaz
Toda la logica interna del lexer, del automata, etc. Expone la interfaz:
```kotlin
interface Lexer {
    fun tokenize(source: String): List<Token>
}
```
## Refactor v1.0 / Inyección de Configuración

### ¿Qué cambiamos?
Antes el Lexer usaba directamente los enums viejos de `domain/Rules.kt` (como `PrintScriptSymbols`, `PrintScriptFunctions`, etc.) para saber qué palabras y símbolos existían.

Ahora el Lexer es 100% genérico y desacoplado del domain:
- Le inyectamos `v1_0keywords` (`Map<String, TokenType>`) con palabras como `"let"`, `"println"`, `"number"`, `"string"`.
- Le inyectamos `v1_0Symbols` (`Map<Char, TokenType>`) con caracteres como `'+'`, `'-'`, `':'`, `';'`, `'='`, `(`, `)`, etc.

### Tokens simplificados
- `Literal`: ahora guarda el valor y su `PSType` directo (`Literal("hola", StrType)` o `Literal("123", NumType)`).
- Como el lexer ya sabe por su autómata si está leyendo comillas (`StringState`) o números (`NumberState`), le asigna el tipo fundamental directamente al comenzar a leer el token
    En el token builder, ya se diferencia el primer caracter agregado al token cuando hace `if(type == null)`. Entonces ahi, y solo ahi, se le pondria el tipo al literal. 
- Si en el futuro agregamos `boolean` (para 1.1), simplemente entra por el mapa de keywords (`"true" to Literal("true", BoolType)`) sin tener que tocar una sola línea del autómata del Lexer.
    En este caso, el boolean `true` o `false` entraria en el camino del identifier, y se construiria como identifier hasta el final, donde se fija si la palabra pertenece al keyword map, y ahi le asigna al token el tipo de esa keyword

## TokenBuilder dinamico

Al lexer le inyectamos mapas de que simbolos/keywords tokenizar:
- Map<String, TokenType> keywords
- Map<String, TokenType> symbols
  
Esto hace que el lexer pase a ser dinamico, funcionando independientemente de que mapas se le pase. Esto hace mas facil la implementación de la nueva version de printscipt, y nuevas versiones futuras.

## Automata

### Versión 3
- Juntamos todos los caracteres que no requerian consumir mas caracteres para ser tokenizados
- Agregamos una rama para tokenizar los 'whitespace' (incluye espacios, tabs y enter) para que nuestro automata no los rechaze (ya que son caracteres aceptados por nuestro vocabulario)
  
  <img width="741" height="467" alt="image" src="https://github.com/user-attachments/assets/eb6edceb-d33a-4ee9-b762-762257287851" />

### Versión 2
- Separamos el type del colon y lo pusimos en la rama de 'keyword'
- Agregamos parentesis en la rama de 'operator'

  <img width="757" height="517" alt="image" src="https://github.com/user-attachments/assets/f48d22e6-1b8c-4027-b100-80acc5fe36ca" />

Ese es el estado actual. ⚠️ Queda pensar bien:
- Tema de los parentesis si lo vamos a dejar así o de otra forma
- Revisar bien que estamos cubriendo cada caso (creo que si)

### Versión 1
Basicamente esta es la idea principal de como "funcionaría" el lexer.
- De forma muy abstracta todavía
- Como no es una opcion simplemente separar por espacios tener algo así nos permite más versatilidad al tokenizar las cosas
  Ejemplo:
  ```let x: number = 5;```
  - el let entraría en la categoría de keyword (es una palabra reservada)
  - la x sería un identifier (nombre de variable)
  - luego el colon ( : ) y el tipo aparecen en la misma rama del automata (siempre despues de un : hay un tipo)
  - el = es un asignment
  - el 5 sería un number literal.
  - por ultimo el semicolon
  Con esto generaríamos una lista de tokens del estilo
```[LET, IDENTIFIER(x), COLON, NUMBER, ASIGNMENT, VALUE(5), SEMICOLON]```

  💡 Basicamente esto sería la salida del lexer. Una lista de tokens:

    <img width="868" height="614" alt="image" src="https://github.com/user-attachments/assets/b0fe42bc-909f-47dd-8a81-ba71213fabc6" />

