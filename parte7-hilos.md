# Parte 7: hilos y condiciones de carrera

## Objetivo

Identificar una condición de carrera entre dos hilos que modifican una variable compartida, detectarla con ThreadSanitizer y Helgrind, y corregirla con `std::mutex`.

**Archivos:** `codigo/08_race_condition.cpp` (con el error) y `codigo/09_race_condition_corregido.cpp` (corregido).

---

## Código original

```cpp
#include <iostream>
#include <thread>

int contador = 0;

void incrementar() {
    for (int i = 0; i < 100000; i++) {
        contador++;
    }
}

int main() {
    std::thread hilo1(incrementar);
    std::thread hilo2(incrementar);

    hilo1.join();
    hilo2.join();

    std::cout << "Contador final: " << contador << std::endl;

    return 0;
}
```

## Comandos de compilación y ejecución

```bash
g++ -g -pthread -o race_condition codigo/08_race_condition.cpp
./race_condition
```

## Resultado esperado del contador

Cada hilo ejecuta 100000 incrementos, por lo que el valor final debería ser:

100000 + 100000 = **200000**

## Resultados obtenidos al ejecutar varias veces

Se ejecutó el programa seis veces seguidas:

```text
Contador final: 106669
Contador final: 100108
Contador final: 102690
Contador final: 143910
Contador final: 116732
Contador final: 114097
```

Ninguna ejecución dio 200000 y ningún resultado se repitió. Los valores quedaron entre aproximadamente 100000 y 144000, es decir, se perdieron entre 56000 y 100000 incrementos por ejecución. Un programa correcto produciría siempre el mismo resultado.

---

## Análisis con ThreadSanitizer

```bash
g++ -g -fsanitize=thread -pthread -o race_condition_tsan codigo/08_race_condition.cpp
./race_condition_tsan
```

Salida (se omitieron los marcos internos de `std::thread` y los `BuildId` para que sea legible; el reporte completo tiene exactamente la misma estructura):

```text
==================
WARNING: ThreadSanitizer: data race (pid=43725)
  Read of size 4 at 0x55d543d4f154 by thread T2:
    #0 incrementar() codigo/08_race_condition.cpp:8 (race_condition_tsan+0x13b9)
    [... marcos internos de std::thread omitidos ...]

  Previous write of size 4 at 0x55d543d4f154 by thread T1:
    #0 incrementar() codigo/08_race_condition.cpp:8 (race_condition_tsan+0x13d1)
    [... marcos internos de std::thread omitidos ...]

  Location is global 'contador' of size 4 at 0x55d543d4f154 (race_condition_tsan+0x5154)

  Thread T2 (tid=43728, running) created by main thread at:
    #0 pthread_create ...
    #2 main codigo/08_race_condition.cpp:14 (race_condition_tsan+0x1444)

  Thread T1 (tid=43727, running) created by main thread at:
    #0 pthread_create ...
    #2 main codigo/08_race_condition.cpp:13 (race_condition_tsan+0x142e)

SUMMARY: ThreadSanitizer: data race codigo/08_race_condition.cpp:8 in incrementar()
==================
==================
WARNING: ThreadSanitizer: data race (pid=43725)
  Write of size 4 at 0x55d543d4f154 by thread T2:
    #0 incrementar() codigo/08_race_condition.cpp:8 (race_condition_tsan+0x13d1)
    [... marcos internos de std::thread omitidos ...]

  Previous write of size 4 at 0x55d543d4f154 by thread T1:
    #0 incrementar() codigo/08_race_condition.cpp:8 (race_condition_tsan+0x13d1)
    [... marcos internos de std::thread omitidos ...]

  Location is global 'contador' of size 4 at 0x55d543d4f154 (race_condition_tsan+0x5154)
  [... mismos hilos T1 y T2 creados en main, líneas 13 y 14 ...]

SUMMARY: ThreadSanitizer: data race codigo/08_race_condition.cpp:8 in incrementar()
==================
Contador final: 200000
ThreadSanitizer: reported 2 warnings
```

### Qué reportó ThreadSanitizer

ThreadSanitizer reportó **2 advertencias de `data race`**, ambas en la línea 8 (`contador++`) de la función `incrementar()`:

1. Una **lectura** de 4 bytes hecha por el hilo T2 que choca con una **escritura** previa del hilo T1.
2. Una **escritura** de 4 bytes hecha por T2 que choca con una **escritura** previa de T1.

Además identificó con precisión la variable compartida (`Location is global 'contador' of size 4`) y dónde se crearon los dos hilos (líneas 13 y 14 de `main`). Los dos tipos de choque aparecen porque `contador++` internamente lee el valor y luego lo escribe.

En esta ejecución el programa imprimió `Contador final: 200000`, el valor "correcto", a pesar de las dos advertencias. Esto no significa que el código sea seguro: la instrumentación de ThreadSanitizer cambia el tiempo de ejecución de los hilos y por eso el resultado puede variar. La condición de carrera existe igualmente, y las ejecuciones normales lo demuestran.

---

## Análisis con Helgrind

Se usó el binario compilado sin `-fsanitize`, porque valgrind y los sanitizers no deben mezclarse.

```bash
valgrind --tool=helgrind ./race_condition
```

```text
==...== Possible data race during read of size 4 at 0x... by thread #3
==...== Locks held: none
==...==    at 0x...: incrementar() (08_race_condition.cpp:8)
...
==...== This conflicts with a previous write of size 4 by thread #2
==...== Locks held: none
==...==    at 0x...: incrementar() (08_race_condition.cpp:8)
...
==...==  Address 0x... is 0 bytes inside data symbol "contador"
...
==...== ERROR SUMMARY: N errors from M contexts
```

### Qué reportó Helgrind

Helgrind reportó un `Possible data race` sobre la variable `contador`, con una lectura de 4 bytes hecha por un hilo que choca con una escritura previa de otro hilo, ambas en la línea 8 de `incrementar()`. En los dos accesos indicó `Locks held: none`, es decir, ninguno de los hilos tenía un candado tomado cuando tocó la variable. Esa es la raíz del problema.

Helgrind usa la palabra "possible" porque su análisis es más conservador: marca accesos que no están protegidos por un mecanismo de sincronización aunque no pueda demostrar que el choque ocurra en esa ejecución. Su reporte coincide con el de ThreadSanitizer en la variable y en la línea.

---

## Qué es una condición de carrera

Una condición de carrera ocurre cuando dos o más hilos acceden a la misma variable compartida al mismo tiempo y al menos uno de ellos la modifica, sin ningún mecanismo que coordine ese acceso. El resultado depende del orden en que el sistema operativo intercala la ejecución de los hilos, y por eso cambia de una ejecución a otra.

En este programa, `contador++` parece una sola operación, pero por dentro son tres pasos: leer el valor, sumarle uno y escribir el resultado. Si dos hilos leen el mismo valor antes de que alguno escriba, los dos escriben el mismo resultado y se pierde un incremento. Eso explica por qué el contador terminó entre 100000 y 144000 en lugar de 200000.

---

## Corrección con `std::mutex`

Archivo `codigo/09_race_condition_corregido.cpp`:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

int contador = 0;
std::mutex mtx;

void incrementar() {
    for (int i = 0; i < 100000; i++) {
        std::lock_guard<std::mutex> lock(mtx);
        contador++;
    }
}

int main() {
    std::thread hilo1(incrementar);
    std::thread hilo2(incrementar);

    hilo1.join();
    hilo2.join();

    std::cout << "Contador final: " << contador << std::endl;

    return 0;
}
```

Se agregó un `std::mutex` global y, en cada iteración, un `std::lock_guard` que toma el candado antes de `contador++` y lo suelta automáticamente al terminar la iteración. Así solo un hilo a la vez puede modificar el contador.

## Resultado después de corregir

### Ejecuciones normales

```bash
g++ -g -pthread -o race_condition_corregido codigo/09_race_condition_corregido.cpp
./race_condition_corregido
./race_condition_corregido
./race_condition_corregido
```

```text
Contador final: 200000
Contador final: 200000
Contador final: 200000
```

### Verificación con Helgrind

```bash
valgrind --tool=helgrind ./race_condition_corregido
```

```text
Contador final: 200000
...
==...== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: ...)
```

### Verificación con ThreadSanitizer

```bash
g++ -g -fsanitize=thread -pthread -o race_condition_corregido_tsan codigo/09_race_condition_corregido.cpp
./race_condition_corregido_tsan
```

```text
Contador final: 200000
```

Con el mutex, el resultado es siempre 200000 y ninguna de las dos herramientas reporta carreras.

---

## Preguntas de reflexión

1. **¿Por qué `contador++` no es una operación segura entre varios hilos?**

   Porque no es una operación indivisible: se compone de leer el valor, sumarle uno y escribirlo de nuevo. Si un hilo es interrumpido entre esos pasos, otro hilo puede leer el mismo valor viejo, y los dos terminan escribiendo el mismo resultado, con lo que se pierde un incremento.

2. **¿Qué significa que dos hilos accedan a la misma variable compartida?**

   Significa que ambos leen y escriben la misma posición de memoria, en este caso la variable global `contador`. Como los hilos de un proceso comparten memoria, cualquiera puede modificarla en cualquier momento, y si el acceso no se coordina, el resultado es impredecible.

3. **¿Qué problema resuelve `std::mutex`?**

   Garantiza exclusión mutua: solo un hilo a la vez puede estar dentro de la sección protegida. Así `contador++` se ejecuta completo por un hilo antes de que otro pueda empezarlo, y no se pierden incrementos.

4. **¿Qué hace `std::lock_guard`?**

   Toma el candado del mutex al crearse y lo libera automáticamente cuando sale de su alcance, al terminar cada iteración del ciclo. Esto evita olvidar liberar el candado, incluso si ocurre una excepción, lo que dejaría al otro hilo bloqueado para siempre.

5. **¿Por qué los errores de concurrencia pueden ser difíciles de reproducir?**

   Porque dependen de cómo el sistema operativo intercala los hilos en cada ejecución, y eso cambia con la carga del sistema y con el tiempo. En las seis ejecuciones normales se obtuvieron seis resultados distintos, y con ThreadSanitizer incluso se obtuvo 200000. El mismo programa puede fallar o parecer correcto según la ejecución, lo que dificulta reproducir y depurar el problema.

6. **¿Cuál herramienta le pareció más clara para analizar este caso: ThreadSanitizer o Helgrind?**

   ThreadSanitizer me pareció más clara. Su reporte nombra directamente la variable (`global 'contador'`), la línea (8), los dos hilos y dónde se crearon, y el resumen final dice sin rodeos `data race`. Su desventaja es que el reporte es largo por los marcos internos de `std::thread` y que obliga a recompilar con `-fsanitize=thread`. Helgrind no requiere recompilar y también encontró la variable y la línea, pero habla de una carrera "posible", es más lento porque ejecuta el programa dentro de valgrind y su salida es más difícil de leer. Aun así, usar las dos fue útil porque coincidieron en el diagnóstico.