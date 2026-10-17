## Parte 3: Introducción a gdb

### Comando de compilación

```bash
g++ -g -o gdb_basico codigo/03_gdb_basico.cpp
```

### Ejecución de gdb

```bash
gdb ./gdb_basico
```

### Explicación de comandos

*   **`-g` (bandera de compilación):** Le indica al compilador que incluya los "símbolos de depuración" en el ejecutable. Esto es lo que permite a gdb leer el código fuente original y saber a qué línea corresponde cada instrucción en la memoria.
*   **`break main`:** Establece un punto de interrupción (*breakpoint*) en la primera línea de la función `main`, ordenándole al depurador que pause el programa justo antes de empezar a ejecutarla.
*   **`run`:** Inicia la ejecución del programa dentro del entorno de gdb.
*   **`next`:** Ejecuta la línea de código actual y avanza a la siguiente en la misma función. Si la línea actual contiene una llamada a una función (como `duplicar`), `next` la ejecuta internamente de principio a fin sin "entrar" en ella paso a paso.
*   **`print`:** Evalúa y muestra el valor actual que tiene una variable en la memoria en el momento exacto en el que el programa está pausado.

### Valores observados

Tras la ejecución línea por línea, se inspeccionaron las variables y se obtuvieron los siguientes valores:
*   `numero`: 5 (antes de asignar, mostraba un valor "basura" porque la memoria no estaba inicializada).
*   `doble`: 10
*   `total`: 20

### Captura de la sesión de gdb

```gdb
(gdb) break main
Breakpoint 1 at 0x1189: file codigo/03_gdb_basico.cpp, line 14.
(gdb) run
Starting program: /ruta/del/laboratorio/codigo/gdb_basico 

Breakpoint 1, main () at codigo/03_gdb_basico.cpp:14
14          int numero = 5;
(gdb) next
15          int doble = duplicar(numero);
(gdb) print numero
$1 = 5
(gdb) next
16          int total = sumar(doble, 10);
(gdb) print doble
$2 = 10
(gdb) next
18          std::cout << "Número: " << numero << std::endl;
(gdb) print total
$3 = 20
(gdb) continue
Continuing.
Número: 5
Doble: 10
Total: 20
[Inferior 1 (process 12345) exited normally]
(gdb) quit
```

### Reflexión

1. **¿Qué es un breakpoint?**
   Es una marca intencional en el código fuente que le indica al depurador que debe pausar la ejecución del programa al llegar a esa línea, permitiendo inspeccionar el estado actual de la memoria y las variables.

2. **¿Qué diferencia hay entre ejecutar el programa normalmente y ejecutarlo dentro de gdb?**
   Ejecutarlo normalmente hace que el programa corra de principio a fin de manera ininterrumpida, mostrando únicamente la salida final en la consola. Al usar gdb, tenemos control total sobre el flujo: podemos detener el tiempo, ver los valores intermedios paso a paso y entender exactamente cómo se van transformando los datos.

3. **¿Qué ventaja tiene inspeccionar variables mientras el programa se ejecuta?**
   Nos permite descubrir errores ocultos. Por ejemplo, al imprimir una variable antes de inicializarla, podemos ver la "basura" en la memoria, lo que nos ayuda a entender el estado exacto del programa y encontrar dónde un cálculo comienza a desviarse de lo esperado.

4. **¿Por qué `next` no entra dentro de las funciones?**
   Porque `next` está diseñado para avanzar secuencialmente en el nivel lógico actual (la función actual). Entiende la llamada a otra función como una sola "caja negra" que debe completar. Esto es útil para saltar funciones que sabemos que funcionan bien y enfocarnos solo en la lógica principal.