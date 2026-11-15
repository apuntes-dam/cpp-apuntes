# 1.5 Entorno: instalar, compilar y ejecutar

## Qué hace falta

| Pieza | Para qué |
|---|---|
| **Un compilador** | Traduce el código a un ejecutable: MSVC (Visual Studio), MinGW-w64/g++, o Clang |
| **Un editor o IDE** | Visual Studio, Visual Studio Code con la extensión *C/C++*, CLion de JetBrains |
| **Un sistema de construcción** (en proyectos más grandes) | CMake es el más usado hoy: describe cómo compilar sin atarte a un IDE concreto |

!!! info "Lo que usa esta web"
    Estas páginas se generan con **MSVC**, el compilador de Microsoft (`cl.exe`), que viene con **Visual Studio** (incluida la edición Community, gratuita) o con las *Build Tools* sueltas, sin instalar el IDE entero. Si no usas Windows, o prefieres herramientas libres multiplataforma, la alternativa más común es **MinGW-w64** (el compilador libre `g++` para Windows) o **Clang**; el código de esta unidad es C++ estándar y compila igual con cualquiera de los tres.

## Comprobar la instalación (MSVC)

MSVC no se añade solo al `PATH`: hace falta abrir una **Developer Command Prompt** (en el menú de Visual Studio) o ejecutar su script de entorno:

```bash
"C:\Program Files\Microsoft Visual Studio\<version>\<edición>\VC\Auxiliary\Build\vcvarsall.bat" x64
cl
```

`cl` sin argumentos muestra su versión y las opciones disponibles: si aparece, el entorno está listo para esa ventana (hay que repetir `vcvarsall.bat` en cada terminal nueva, o usar directamente la Developer Command Prompt).

## Compilar y ejecutar un archivo suelto

```bash
cl /EHsc /std:c++20 hola.cpp
hola.exe
```

| Opción | Qué hace |
|---|---|
| `/EHsc` | Activa el manejo de excepciones de C++ (casi siempre se quiere) |
| `/std:c++20` | Usa el estándar C++20 (sin indicarlo, MSVC usa una versión más antigua por defecto) |
| `/Fe:nombre.exe` | Elige el nombre del ejecutable (si no, usa el del primer `.cpp`) |

Con MinGW-w64/g++ el equivalente es `g++ -std=c++20 -Wall hola.cpp -o hola`; con Clang, `clang++ -std=c++20 hola.cpp -o hola`.

## Un proyecto con CMake (opcional, para ir más allá de un archivo)

```text
mi_proyecto/
├── CMakeLists.txt
└── main.cpp
```

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.20)
project(mi_proyecto)
set(CMAKE_CXX_STANDARD 20)
add_executable(mi_proyecto main.cpp)
```

```bash
cmake -B build
cmake --build build
```

CMake genera los archivos de proyecto para el compilador que tengas (MSVC, Makefiles de g++...) a partir de una descripción que no depende del sistema operativo. Para un solo archivo de prácticas no hace falta; para un proyecto con varios archivos y librerías, ahorra mucho trabajo.

!!! note "Errores frecuentes"
    * `'cl' is not recognized`: no se ha ejecutado `vcvarsall.bat` en esa terminal (o no se ha abierto la Developer Command Prompt).
    * `LNK2019: unresolved external symbol`: falta compilar o enlazar algún `.cpp`, o se olvidó `#include` de la declaración.
    * El compilador avisa de una variable sin usar: es un aviso (`warning`), no un error; el programa compila igual.
