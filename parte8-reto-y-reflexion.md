## Reflexión final

1. **¿Qué entiende ahora por depuración?**

   La depuración es el proceso de encontrar, entender y corregir errores en un programa observando cómo se comporta mientras se ejecuta. No consiste en adivinar ni en cambiar código al azar: se reproduce el problema, se usa una herramienta para ver qué ocurre realmente, se identifica la causa, se corrige y se verifica que el problema desapareció.

2. **¿Por qué no basta con que un programa compile?**

   Porque compilar solo comprueba que el código sea válido según las reglas del lenguaje. Un programa puede compilar sin errores y aun así dar resultados incorrectos (como el promedio que devolvía 270), perder memoria, escribir fuera de un arreglo o perder incrementos entre hilos. Todos esos errores aparecieron en el laboratorio con programas que compilaban sin ninguna advertencia.

3. **¿Cuál fue la diferencia entre un error de sintaxis y un error lógico?**

   El error de sintaxis impide que el programa se compile y el compilador lo señala con el archivo, la línea y la causa. El error lógico no impide nada: el programa compila, se ejecuta y produce un resultado, pero ese resultado no es el esperado. Por eso el segundo es más difícil de encontrar: hay que comparar lo obtenido contra lo esperado, porque nadie avisa dónde está el problema.

4. **¿Qué ventaja tiene usar gdb?**

   Permite detener el programa en un punto, avanzar línea por línea y ver los valores reales de las variables en cada momento. En la Parte 4, por ejemplo, permitió ver que `divisor` valía 0 antes de entrar a `dividir`, y `backtrace` mostró toda la cadena de llamadas que llevó hasta el fallo. Es la herramienta más útil cuando hay que seguir el razonamiento del programa paso a paso.

5. **¿Qué ventaja tiene usar valgrind?**

   No requiere recompilar y detecta errores de memoria que el programa no muestra por sí solo: fugas, accesos fuera de límites y uso de memoria liberada. En la Parte 5, los dos programas parecían funcionar bien y valgrind reportó exactamente qué línea reservó la memoria perdida y qué línea escribió fuera del arreglo. Además, con Helgrind permite analizar problemas de hilos. Su desventaja es que hace el programa mucho más lento.

6. **¿Qué ventaja tiene usar AddressSanitizer?**

   Detecta errores de memoria en tiempo de ejecución con reportes muy claros, que indican dónde se usó, dónde se liberó y dónde se reservó cada bloque, como en el `heap-use-after-free` de la Parte 6. Es más rápido que valgrind, pero requiere recompilar con `-fsanitize=address` y detiene el programa en el primer error. Lo más valioso es que convierte un error silencioso en un fallo visible y fácil de localizar.

7. **¿Qué aprendió sobre errores de memoria?**

   Que son peligrosos justamente porque muchas veces no producen un fallo visible: el programa con la fuga terminó bien, el que escribía fuera del arreglo imprimió su mensaje