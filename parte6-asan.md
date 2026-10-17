## Parte 6: AddressSanitizer

### Resultados de Ejecución

**Qué ocurrió al ejecutar sin sanitizer:**
Al ejecutar el programa compilado de forma normal (`./use_after_free`), el programa imprimió `Número: -871267837` y finalizó sin mostrar ningún error visible ni interrupción, lo cual oculta el hecho de que se accedió a memoria inválida.

**Qué ocurrió al ejecutar con AddressSanitizer:**
AddressSanitizer detectó inmediatamente el error de memoria en tiempo de ejecución, abortó el programa (`==38329==ABORTING`) y generó un reporte detallado (stack trace) mostrando exactamente dónde se reservó la memoria, dónde se liberó y dónde se intentó acceder incorrectamente a ella.

**Qué significa `heap-use-after-free`:**
Significa que el programa intentó leer o escribir en una dirección de memoria ubicada en el *heap* (memoria dinámica) que ya había sido devuelta al sistema (liberada) mediante la instrucción `delete`. Es un acceso a memoria "muerta" o no válida.

**Línea donde ocurre el error:**
Según el reporte (`#0 0x5e505a7b934d in main codigo/07_use_after_free.cpp:9`), el error (la lectura inválida) ocurre en la línea 9:
`std::cout << "Número: " << *numero << std::endl;`

**Código corregido:**
Se movió la instrucción de impresión para que se ejecute *antes* de liberar la memoria, y se agregó una buena práctica de seguridad asignando `nullptr` al puntero después del `delete`:

```cpp
#include <iostream>

int main() {
    int* numero = new int;
    *numero = 42;

    // Corrección: Se movió la impresión antes del delete
    std::cout << "Número: " << *numero << std::endl; 

    delete numero;
    // Corrección: Evitar que el puntero quede "colgando" (dangling pointer)
    numero = nullptr; 

    return 0;
}
```

**Resultado final:**
Al volver a compilar y ejecutar con ASan (`./use_after_free_asan`), el programa imprime `Número: 42` y finaliza limpiamente sin arrojar ningún reporte ni mensaje de error de AddressSanitizer.

### Preguntas de Reflexión

1. **¿Qué significa usar memoria después de liberarla?**
   Significa intentar acceder al contenido de una dirección de memoria a través de un puntero (*dangling pointer*) después de haber invocado `delete` sobre él. Para el sistema operativo, ese bloque de memoria ya no pertenece al programa y su contenido es considerado basura o puede haber sido reasignado a otro proceso.

2. **¿Por qué este error puede ser difícil de detectar sin herramientas?**
   Porque, a menudo, los datos antiguos permanecen en la memoria física por un breve tiempo después de liberarlos. Si el programa intenta leerlos de inmediato (como sucedió al ejecutarlo sin ASan), es probable que encuentre el valor "correcto" (42) y no ocurra un *Segmentation Fault*, dando la falsa impresión de que el programa funciona perfectamente hasta que el fallo ocurre aleatoriamente en un momento crítico.

3. **¿Qué ventaja tiene AddressSanitizer sobre ejecutar el programa normalmente?**
   ASan instrumenta el código directamente durante la compilación, rodeando las variables con zonas de memoria "envenenadas" (redzones). Esto le permite atrapar los accesos inválidos en el acto y abortar la ejecución inmediatamente con un reporte exacto de la línea de código, impidiendo que el error pase desapercibido.

4. **¿Qué diferencia observó entre el reporte de `valgrind` y el reporte de AddressSanitizer?**
   AddressSanitizer es significativamente más rápido porque su instrumentación se integra al compilar el binario, mientras que `valgrind` (Memcheck) ejecuta el programa en un entorno emulado, lo que lo hace más lento. Además, ASan requiere recompilar el código fuente con la bandera `-fsanitize=address`, mientras que Valgrind puede analizar cualquier binario precompilado sin requerir modificaciones.

5. **¿Por qué es buena práctica asignar `nullptr` después de liberar un puntero?**
   Porque anula explícitamente el puntero. Si accidentalmente el programa intenta usar ese puntero más adelante en el código, desreferenciar un `nullptr` provocará un fallo inmediato, claro y predecible (*Segmentation Fault* apuntando a la línea exacta), en lugar de causar una corrupción de memoria silenciosa e impredecible (*use-after-free*).