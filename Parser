### Aparición de un automata nuevo para el parser

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
