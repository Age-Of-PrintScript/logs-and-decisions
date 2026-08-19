# Historial de decisiones
## Idea Inicial
El formatter va a usar la salida del parser para ver la estructura del programa. En base a eso, va a reesribir el código fuente, pero con un formato especifico.
Por ende, va a tener que reconstruir cada linea de código dentro de un archivo. Cada linea seria un AST, y en base a que tipo es, que palabras se agregan (por ejemplo el Let en la Declaration). Ademas, el formatter se va a tener que asegurar que ciertas reglas sean cumplidas. 

Algunas son configurables, y para eso se va a definir un json mas o menos asi:
```
rules {
  space_before_colon: true,
  space_after_colon: true,
  spaces_around_assign: true,
  lines_before_call: {0, 1, 2}
}
```
Otras son obligatorias, como:
- Cada linea debe terminar en un ;
- Despues de un ; debe haber un salto de linea
- Antes y despues de cada operador debe haber un espacio
