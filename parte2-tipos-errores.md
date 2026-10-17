g# Parte 2: tipos de errores

## Objetivo

Distinguir entre errores de sintaxis, errores de tiempo de ejecución y errores lógicos, observando cómo se manifiesta cada uno y qué herramienta lo detecta.

---

## Ejercicio 1: error de sintaxis

**Archivo:** `codigo/01_error_sintaxis.cpp`

### Código original

```cpp
#include <iostream>

int main() {
    std::cout << "Hola mundo" << std::endl
    return 0;
}
```

### Comandos de compilación y ejecución

```bash
g++ -g -o error_sintaxis codigo/01_error_sintaxis.cpp
./error_sintaxis
```

### Mensaje de error original

```text
codigo/01_error_sintaxis.cpp: In function ‘int main()’:
codigo/01_error_sintaxis.cpp:4:43: error: expected ‘;’ before ‘return’
    4 |     std::cout << "Hola mundo" << std::endl
      |                                           ^
      |                                           ;
    5 |     return 0;
      |     ~~~~~~
```

La compilación falló y no se generó el ejecutable, por lo que no fue posible ejecutar el programa.

### Línea donde estaba el error

El compilador reporta la línea 4, columna 43 del archivo `codigo/01_error_sintaxis.cpp`, justo al final de la instrucción `std::cout << "Hola mundo" << std::endl`.

### Causa del error

Falta el punto y coma (`;`) que debe terminar la instrucción. Como C++ no reconoce el fin de la instrucción, interpreta que continúa en la línea siguiente y reporta el problema al encontrar `return`.

### Código corregido

```cpp
#include <iostream>

int main() {
    std::cout << "Hola mundo" << std::endl;
    return 0;
}
```

### Resultado de la ejecución corregida

```bash
g++ -g -o error_sintaxis codigo/01_error_sintaxis.cpp
./error_sintaxis
```

```text
Hola mundo
```

### Preguntas de reflexión

1. **¿Este error fue detectado antes o durante la ejecución?**

   Fue detectado antes de la ejecución, durante la compilación. Como el compilador no pudo traducir el código, ni siquiera se generó un ejecutable que se pudiera correr.

2. **¿Qué herramienta detectó el error?**

   El compilador `g++`.

3. **¿Por qué este tipo de error suele ser más fácil de corregir que un error lógico?**

   Porque el compilador indica el archivo, la línea, la columna y la causa probable (en este caso, que esperaba un `;`). En un error lógico el programa compila y corre, así que nadie señala dónde está el problema y hay que descubrirlo comparando resultados.

---

## Ejercicio 2: error lógico

**Archivo:** `codigo/02_error_logico.cpp`

### Código original

```cpp
#include <iostream>
#include <vector>

double calcular_promedio(const std::vector<int>& notas) {
    int suma = 0;

    for (int nota : notas) {
        suma += nota;
    }

    return suma;
}

int main() {
    std::vector<int> notas = {80, 90, 100};

    double promedio = calcular_promedio(notas);

    std::cout << "Promedio: " << promedio << std::endl;

    return 0;
}
```

### Comandos de compilación y ejecución

```bash
g++ -g -o error_logico codigo/02_error_logico.cpp
./error_logico
```

### Resultado obtenido originalmente

El programa compila sin errores y se ejecuta, pero imprime:

```text
Promedio: 270
```

### Resultado esperado

Con las notas 80, 90 y 100, el promedio es:

(80 + 90 + 100) / 3 = 270 / 3 = **90**

El programa mostró 270, que es la suma de las notas, en lugar del promedio real.

### Explicación del error

La función `calcular_promedio` suma correctamente todas las notas en la variable `suma`, pero termina con `return suma;` sin dividir entre la cantidad de notas. Por eso devuelve la suma y no el promedio.

El compilador no lo detecta porque el código es válido: devolver un `int` en una función que retorna `double` es una conversión permitida. El error está en la lógica del cálculo, no en la forma de escribirlo.

### Código corregido

```cpp
#include <iostream>
#include <vector>

double calcular_promedio(const std::vector<int>& notas) {
    int suma = 0;

    for (int nota : notas) {
        suma += nota;
    }

    return static_cast<double>(suma) / notas.size();
}

int main() {
    std::vector<int> notas = {80, 90, 100};

    double promedio = calcular_promedio(notas);

    std::cout << "Promedio: " << promedio << std::endl;

    return 0;
}
```

Se usa `static_cast<double>(suma)` para que la división sea decimal y no se pierdan los decimales, como ocurriría con una división entera entre `int`.

### Resultado final

```bash
g++ -g -o error_logico codigo/02_error_logico.cpp
./error_logico
```

```text
Promedio: 90
```

### Preguntas de reflexión

1. **¿Por qué el compilador no detectó este error?**

   Porque el compilador solo verifica que el código sea válido según las reglas del lenguaje. El código era válido, y el compilador no puede saber que el programador quería calcular un promedio y no una suma.

2. **¿Por qué este error se considera lógico?**

   Porque el programa compila, se ejecuta sin fallar y produce un resultado, pero ese resultado no es el que se esperaba. El error está en el razonamiento del algoritmo (falta la división) y no en la sintaxis ni en la ejecución.

3. **¿Qué estrategia usó para encontrarlo?**

   Calculé a mano el resultado esperado (90) y lo comparé con el que imprimió el programa (270). Como la diferencia era evidente, revisé la función y noté que devolvía la suma sin dividir.

4. **¿Qué pruebas adicionales podría hacer?**

   Probar con otras notas cuyo promedio no sea exacto, como `{80, 90, 95}` (para verificar que no haya división entera), con una sola nota, con notas iguales y con un vector vacío. Este último caso divide entre cero, así que habría que decidir cómo manejarlo (por ejemplo, devolver 0 o avisar del error).