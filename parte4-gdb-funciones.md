## Parte 4: step, next y backtrace

### Error observado al ejecutar normalmente
Al ejecutar el programa original `./gdb_funciones` sin herramientas de depuración, el programa compila exitosamente, pero al correrlo se interrumpe abruptamente con un error del sistema operativo, típicamente mostrando: `Floating point exception (core dumped)` o una excepción aritmética.

### Hallazgos usando gdb
Ejecutando el programa paso a paso, se inspeccionaron las variables y se encontraron los siguientes valores en la memoria:
* `valor` (en `main`): 5
* `x` (parámetro en `calcular`): 5
* `divisor` (calculado en `calcular`): 0
* `b` (parámetro en `dividir`): 0

**Sesión de gdb:**
```text
(gdb) break main
Breakpoint 1 at 0x1189: file codigo/04_gdb_funciones.cpp, line 14.
(gdb) run
Starting program: /ruta/codigo/gdb_funciones 

Breakpoint 1, main () at codigo/04_gdb_funciones.cpp:14
14          int valor = 5;
(gdb) next
15          int respuesta = calcular(valor);
(gdb) step
calcular (x=5) at codigo/04_gdb_funciones.cpp:9
9           int divisor = x - 5;
(gdb) step
10          int resultado = dividir(100, divisor);
(gdb) print x
$1 = 5
(gdb) print divisor
$2 = 0
(gdb) step
dividir (a=100, b=0) at codigo/04_gdb_funciones.cpp:4
4           int resultado = a / b;
(gdb) print a
$3 = 100
(gdb) print b
$4 = 0
(gdb) backtrace
#0  dividir (a=100, b=0) at codigo/04_gdb_funciones.cpp:4
#1  0x0000555555555169 in calcular (x=5) at codigo/04_gdb_funciones.cpp:10
#2  0x000055555555518b in main () at codigo/04_gdb_funciones.cpp:15
(gdb) next
Program received signal SIGFPE, Arithmetic exception.
0x000055555555513b in dividir (a=100, b=0) at codigo/04_gdb_funciones.cpp:4
4           int resultado = a / b;
(gdb) quit
```

### Código corregido

Se modificó la función `dividir` para validar el parámetro de entrada `b` y evitar la división por cero:

```cpp
#include <iostream>

int dividir(int a, int b) {
    if (b == 0) {
        std::cerr << "Error: división entre cero" << std::endl;
        return 0;
    }

    int resultado = a / b;
    return resultado;
}

int calcular(int x) {
    int divisor = x - 5;
    int resultado = dividir(100, divisor);
    return resultado;
}

int main() {
    int valor = 5;
    int respuesta = calcular(valor);

    std::cout << "Respuesta: " << respuesta << std::endl;

    return 0;
}
```

### Resultado de la ejecución corregida
```text
Error: división entre cero
Respuesta: 0
```

### Reflexión

1. **¿Qué diferencia observó entre `next` y `step`?**
   Mientras que `next` ejecuta una función completa como si fuera una sola instrucción (saltando a la siguiente línea del contexto actual), `step` "entra" dentro de la función llamada, permitiendo depurar su interior línea por línea.

2. **¿Para qué sirve `backtrace`?**
   Muestra la "pila de llamadas" (call stack), indicando exactamente qué función llamó a qué función, en qué orden y con qué parámetros, hasta llegar a la instrucción actual donde se detuvo el programa.

3. **¿Cuál fue la causa del error?**
   La función `calcular` resta 5 a la variable de entrada `x` (que valía 5). Esto produjo que la variable `divisor` fuera 0. Este 0 se pasó como parámetro `b` a la función `dividir`, provocando una operación matemática indefinida (100 / 0).

4. **¿Por qué este error es de tiempo de ejecución?**
   Porque la sintaxis de `a / b` es perfectamente válida para el compilador. El error solo se manifiesta cuando el programa está corriendo y las variables toman valores específicos (en este caso, cuando `b` se convierte en 0) que el procesador no puede resolver matemáticamente.

5. **¿Cómo podría prevenirse este tipo de error desde el diseño del programa?**
   Se debe validar la entrada de datos antes de realizar operaciones críticas. En lugar de devolver un `0` (que podría confundirse con un resultado válido real), se podría prevenir por diseño lanzando una excepción (`throw std::invalid_argument`), usando tipos de datos seguros como `std::optional<int>` o devolviendo códigos de error específicos.