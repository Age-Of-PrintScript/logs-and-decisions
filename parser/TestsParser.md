# Flujo general
Hay dos tipos de tests: uno principal que testea el parser completo, y otro que testea la resolución de expresiones aritméticas por separado.
## Tests parser end-to-end
Cada test vive en un archivo .md dentro de parser/src/test/resources/parserTests/ 

El framework lee todos los archivos, y cada archivo = un test dinámico.

Cada .md tiene dos secciones:
1. \# Input — una lista de tokens descritos en texto, uno por línea. Por ejemplo:
   
   ```
    LET 
    IDENTIFIER: X
    COLON
    TYPE: NUMBER
    ASSIGN
    NUMBER LITERAL: 5
    SEMICOLON
   ```
3. \# Expected: SUCCESS  o  \# Expected: FAILURE — el AST esperado o el nombre de un error (ej: MISSING_IDENTIFIER).
      ```
         DECLARATION
            X
            NUMBER
            LITERAL NUMBER 5
      ```

El flujo de cada test es:
Tokens en texto → Token reales → se pasan al Parser → se compara el resultado (AST o SyntaxError) contra lo esperado.
No se usa el lexer. Los tests le llegan directo como tokens al parser, para aislarlo.

## Tests del expression solver
Estos tests son independientes. Construyen objetos Expression (nodos del AST) sin pasar por tokens ni parser, y verifica que el expression solver devuelva el número correcto.

## Convenciones clave
- Separar datos de lógica: agregar un test nuevo es solo crear un .md, no tocar código.
- Comparación estructural completa: se compara todo el AST, no solo partes.
- Posiciones dummy: todos los tokens de test tienen posición (0,0) porque no se testea eso.
- Errores como enum: los casos de fallo solo nombran el valor de SyntaxError esperado.
