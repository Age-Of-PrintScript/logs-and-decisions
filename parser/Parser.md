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
  
