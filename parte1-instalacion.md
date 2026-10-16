# Parte 1: instalación y verificación del ambiente

## Objetivo

Verificar que el ambiente de trabajo tiene las herramientas necesarias para compilar, ejecutar y depurar programas en C++.

## Sistema operativo utilizado

```bash
lsb_release -a
uname -a
```

```text
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 26.04.1 LTS
Release:	26.04
Codename:	resolute
Linux gulloab-IdeaPad-3-15ITL6 7.0.0-38-generic #38-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep  4 09:10:14 UTC 2026 x86_64 GNU/Linux
```

El laboratorio se realiza en Ubuntu 26.04.1 LTS instalado directamente en la máquina (sin WSL2 ni máquina virtual), con arquitectura x86_64.

## Instalación de las herramientas

```bash
sudo apt update
sudo apt install build-essential gdb valgrind
```

```text
Obj:1 http://security.ubuntu.com/ubuntu resolute-security InRelease
Obj:2 http://archive.ubuntu.com/ubuntu resolute InRelease
Obj:3 http://archive.ubuntu.com/ubuntu resolute-updates InRelease
Obj:4 http://archive.ubuntu.com/ubuntu resolute-backports InRelease
Se pueden actualizar 8 paquetes. Ejecute «apt list --upgradable» para verlos.
build-essential ya está en su versión más reciente (12.12ubuntu2.26.04.2).
gdb ya está en su versión más reciente (17.1-2ubuntu1).
fijado gdb como instalado manualmente.
valgrind ya está en su versión más reciente (1:3.26.0-0ubuntu1).
Resumen:
  Actualizando: 0, Instalando 0, Eliminando: 0, no actualizando: 8
```

Las tres herramientas ya estaban instaladas, por lo que no se instaló ningún paquete nuevo.

## Versiones de las herramientas

### g++

```bash
g++ --version
```

```text
g++ (Ubuntu 15.2.0-16ubuntu1) 15.2.0
Copyright (C) 2025 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

### gdb

```bash
gdb --version
```

```text
GNU gdb (Ubuntu 17.1-2ubuntu1) 17.1
Copyright (C) 2025 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

### valgrind

```bash
valgrind --version
```

```text
valgrind-3.26.0
```

## Programa de prueba

Archivo `codigo/prueba.cpp`:

```cpp
#include <iostream>

int main() {
    std::cout << "Ambiente listo para depuración" << std::endl;
    return 0;
}
```

### Compilación y ejecución

```bash
g++ -g -o prueba codigo/prueba.cpp
./prueba
```

```text
Ambiente listo para depuración
```

El programa compiló sin errores ni advertencias y se ejecutó correctamente, lo que confirma que el ambiente está listo.

## Para qué sirve cada herramienta

- **`g++`:** es el compilador de C++ de GNU. Traduce el código fuente a un ejecutable que el procesador puede correr.
- **`gdb`:** es el depurador de GNU. Permite ejecutar un programa de forma controlada: detenerlo en puntos específicos, avanzar línea por línea, inspeccionar variables y revisar la pila de llamadas.
- **`valgrind`:** es un conjunto de herramientas de análisis dinámico. Con Memcheck detecta errores de memoria como fugas, accesos inválidos y uso de memoria liberada, y con Helgrind detecta problemas de concurrencia.

## Preguntas de reflexión

1. **¿Para qué sirve `g++`?**

   Sirve para compilar programas escritos en C++ y generar el ejecutable. También reporta los errores de sintaxis y las advertencias del código.

2. **¿Para qué sirve `gdb`?**

   Sirve para observar y controlar un programa mientras se ejecuta, de modo que se pueda encontrar en qué punto y por qué falla o produce resultados incorrectos.

3. **¿Para qué sirve `valgrind`?**

   Sirve para analizar un programa en ejecución y detectar errores de memoria (fugas, accesos fuera de límites, uso de memoria liberada) y de concurrencia, incluso cuando el programa parece funcionar bien.

4. **¿Por qué se recomienda compilar con `-g` al depurar?**

   Porque agrega al ejecutable la información de depuración: nombres de variables y funciones, y la correspondencia entre el código máquina y las líneas del código fuente. Sin ella, `gdb` y `valgrind` solo mostrarían direcciones de memoria en lugar de nombres y números de línea.

5. **¿Qué diferencia hay entre compilar un programa y depurarlo?**

   Compilar es traducir el código fuente a un ejecutable y detectar errores de sintaxis antes de que el programa corra. Depurar es analizar el programa mientras se ejecuta para encontrar y entender errores de ejecución o lógicos que el compilador no puede detectar.