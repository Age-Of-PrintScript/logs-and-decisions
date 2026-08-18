# Historial de decisiones
## Automata
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

💡 Basicamente esto sería la salida del lexer. Una lista de tokens



  <img width="868" height="614" alt="image" src="https://github.com/user-attachments/assets/b0fe42bc-909f-47dd-8a81-ba71213fabc6" />
