# Flujo general
Hay dos suites de tests: una principal que testea el parser completo, y otra que testea la resolución de expresiones aritméticas por separado.
## Suite principal: tests data-driven con archivos markdown
Cada test vive en un archivo .md dentro de parser/src/test/resources/parserTests/ (hay 30). El framework lee todos los archivos, y cada archivo = un test dinámico.
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
3. \# Expected: SUCCESS o \# Expected: FAILURE — el AST esperado (con notación de indentación) o el nombre de un error (ej: MISSING_IDENTIFIER).

El flujo de cada test es:
Tokens en texto → Token reales → se pasan al Parser → se compara el resultado (AST o SyntaxError) contra lo esperado.
No se usa el lexer. Los tests le llegan directo como tokens al parser, para aislarlo.

## Suite secundaria: tests del solver de expresiones
Esta suite es independiente. Construye objetos Expression (nodos del AST) directamente en Kotlin sin pasar por tokens ni parser, y verifica que el evaluador aritmético devuelva el número correcto.

Conveniones clave
- Separar datos de lógica: agregar un test nuevo es solo crear un .md, no tocar código.
- Comparación estructural completa: se compara todo el AST, no solo partes.
- Posiciones dummy: todos los tokens de test tienen posición (0,0) porque no se testea eso.
- Errores como enum: los casos de fallo solo nombran el valor de SyntaxError esperado.
En resumen: los tests prueban el parser alimentándole tokens directamente (sin lexer) y comparando el AST resultado contra una representación textual en markdown. Los errores se validan matcheando contra un enum. Los tests de expresiones son un mundo aparte que evalúa aritmética construyendo nodos del AST a mano.
▣  Plan · Big Pickle · 2m 25s
