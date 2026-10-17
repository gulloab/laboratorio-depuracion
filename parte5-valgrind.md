## Parte 5: Análisis de memoria con Valgrind

### Ejercicio 1: Pérdida de memoria (Memory Leak)

**Qué hace el programa:**
El programa reserva dinámicamente un arreglo de 10 enteros en el *heap* (memoria dinámica), asigna valores a cada posición usando un ciclo e imprime el primer elemento.

**Error de memoria:**
El programa nunca libera la memoria reservada con `new[]` antes de terminar su ejecución, lo que genera una fuga o pérdida de memoria (*memory leak*).

**Reporte de Valgrind (original):**
`valgrind` reporta un error de memoria perdida, indicando que hay bloques asignados que no se liberaron. Específicamente, muestra `definitely lost: 40 bytes in 1 blocks` (10 enteros de 4 bytes cada uno).

**Qué significa `definitely lost`:**
Significa que el bloque de memoria fue reservado, pero el programa perdió todos los punteros que apuntaban a esa dirección antes de liberarla (o el puntero salió de su alcance - *scope*). Es memoria que ya no se puede utilizar ni recuperar durante la ejecución del programa.

**Corrección:**
Se agregó la instrucción para liberar el arreglo dinámico al final de la función `crear_arreglo()`:
```cpp
    std::cout << "Primer elemento: " << datos[0] << std::endl;
    delete[] datos; // Corrección
}
```

**Resultado de Valgrind después de corregir:**
```text
All heap blocks were freed -- no leaks are possible
ERROR SUMMARY: 0 errors from 0 contexts
```

#### Preguntas de reflexión (Ejercicio 1)

1. **¿Qué es una pérdida de memoria?**
   Es un error de software que ocurre cuando un programa reserva memoria dinámica (en el *heap*) y no la devuelve al sistema operativo una vez que deja de utilizarla, haciendo que esa memoria quede inaccesible y desperdiciada.
2. **¿Por qué el programa puede terminar aparentemente bien aunque tenga una pérdida de memoria?**
   Porque al finalizar el proceso del programa, el sistema operativo moderno interviene y reclama (libera) toda la memoria que se le había asignado al proceso de forma automática. El problema grave ocurre mientras el programa sigue corriendo.
3. **¿Qué significa liberar memoria dinámica?**
   Significa notificarle al administrador de memoria del sistema operativo que el programa ya terminó de usar un bloque específico de memoria, permitiendo que ese espacio sea reasignado para otros usos u otros programas.
4. **¿Por qué se usa `delete[]` y no solo `delete`?**
   Porque se está liberando un arreglo. `delete[]` le indica al compilador que debe invocar los destructores de todos los elementos del arreglo antes de liberar el bloque completo. Usar solo `delete` en un arreglo resulta en comportamiento indefinido.
5. **¿Qué tipo de problemas podrían aparecer en programas grandes si no se libera memoria?**
   El programa consumirá progresivamente más memoria RAM con el tiempo. Esto puede hacer que el sistema operativo se vuelva lento (al usar memoria virtual en disco) y, eventualmente, el programa colapsará al quedarse sin memoria disponible (error *Out of Memory*).

---

### Ejercicio 2: Acceso fuera de límites (Invalid Access)

**Resultado de ejecutar normalmente:**
El programa ejecuta y llega a imprimir `Programa finalizado` sin que ocurra un choque evidente (sin *Segmentation Fault*), lo que oculta el error.

**Resultado con Valgrind:**
```text
Invalid write of size 4
   at 0x...: main (06_invalid_access.cpp:7)
 Address 0x... is 0 bytes after a block of size 20 alloc'd
   at 0x...: operator new[](unsigned long) (vg_replace_malloc.c:...)
   by 0x...: main (06_invalid_access.cpp:4)
```

**Qué significa `Invalid write`:**
Significa que el programa intentó escribir datos (de tamaño 4 bytes, es decir, un `int`) en una dirección de memoria que no le pertenece o que no fue reservada para ese propósito (en este caso, 0 bytes después del bloque de 20 bytes que sí fue reservado).

**Línea donde ocurre el problema:**
Ocurre en la línea 7: `arreglo[i] = i * 10;`. En la última iteración, `i` vale 5, por lo que el programa intenta acceder a `arreglo[5]`, pero los índices válidos para un arreglo de tamaño 5 son del 0 al 4.

**Código corregido:**
Se cambió la condición del ciclo `for` de `<=` a `<` para evitar acceder al sexto elemento inexistente:
```cpp
    for (int i = 0; i < 5; i++) { // Corrección
        arreglo[i] = i * 10;
    }
```

**Resultado final con Valgrind:**
```text
ERROR SUMMARY: 0 errors from 0 contexts
```

#### Preguntas de reflexión (Ejercicio 2)

1. **¿Por qué el programa podría no fallar aunque acceda fuera del arreglo?**
   Porque el espacio de memoria inmediatamente posterior al arreglo podría pertenecer a la misma página de memoria que el sistema operativo asignó al proceso. Al no violar los permisos de acceso del sistema operativo, este no emite una señal de terminación (SIGSEGV), aunque el programa esté dañando sus propios datos internos.
2. **¿Qué significa escribir fuera de los límites de un arreglo?**
   Significa calcular una dirección de memoria utilizando un índice que excede la capacidad declarada del arreglo, e intentar almacenar información allí, sobrescribiendo bytes que pertenecen a otras variables o a metadatos de control del *heap*.
3. **¿Por qué este tipo de error es peligroso?**
   Porque causa corrupción de memoria silenciosa. Altera variables impredecibles del programa, lo que resulta en fallos lógicos muy difíciles de rastrear o abre vulnerabilidades de seguridad graves (como el *buffer overflow*).
4. **¿Qué diferencia hay entre un error visible y un comportamiento indefinido?**
   Un error visible produce un fallo inmediato y predecible (como un cierre abrupto). El comportamiento indefinido significa que el estándar de C++ no garantiza qué pasará: el programa podría funcionar bien hoy, fallar mañana, dar un resultado incorrecto, o comportarse de forma totalmente caótica según el compilador o la máquina.