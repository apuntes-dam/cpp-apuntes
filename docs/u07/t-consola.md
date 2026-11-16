# 7.A Consola: entrada y salida

## Los tres flujos

| Flujo | En C++ | Para qué |
|---|---|---|
| Entrada estándar | `std::cin` | Lo que escribe el usuario |
| Salida estándar | `std::cout` | La salida normal del programa |
| Salida de error | `std::cerr` | Avisos y errores, **separados** de la salida normal |

Separar la salida normal de la de error permite redirigir cada una a su sitio: `programa > resultado.txt 2> errores.txt` guarda una en cada archivo.

```cpp
#include <iostream>

int main() {
    std::cout << "Esto va a la salida normal" << std::endl;
    std::cerr << "Esto va a la salida de error" << std::endl;

    std::cout << "Sin salto de línea... ";
    std::cout.flush();      // por si la salida se queda retenida en el búfer
    std::cout << "ahora sí" << std::endl;
    return 0;
}
```

Salida:

```text
Esto va a la salida normal
Sin salto de línea... ahora sí
```

## Formato al mostrar

```cpp
#include <bitset>
#include <iomanip>
#include <iostream>

int main() {
    std::string nombre = "Pan";
    double precio = 1.2;
    int unidades = 5;

    std::cout << std::left << std::setw(10) << nombre << std::right
              << std::fixed << std::setprecision(2) << std::setw(6) << precio
              << std::setw(5) << unidades << '\n';                     // alinear a la izquierda, derecha, con decimales

    std::cout << std::setfill('0') << std::setw(8) << std::setprecision(3) << 3.14159 << std::setfill(' ') << '\n';   // relleno con ceros

    std::cout << std::showpos << 5 << std::noshowpos << '\n';           // con signo siempre

    std::cout << std::showbase << std::hex << 255 << std::dec << " " << std::bitset<8>(5) << '\n';   // hexadecimal y binario
    return 0;
}
```

Salida:

```text
Pan         1.20    5
0003.142
+5
0xff 00000101
```

`std::fixed`, `std::setw` y los demás manipuladores de `<iomanip>` quedan **fijados** en el flujo hasta que los cambias: a diferencia de un `{:.2f}` de otros lenguajes, aquí el formato es un estado que se va acumulando línea a línea.

## Argumentos de la línea de comandos

Un programa puede recibir datos al lanzarlo, sin esperar a que el usuario los escriba:

```cpp
#include <iostream>

int main(int argc, char* argv[]) {
    std::cout << argc << " argumentos" << std::endl;     // el primero siempre es el nombre del programa
    for (int i = 0; i < argc; i++) {
        std::cout << i << ": " << argv[i] << std::endl;
    }
    return 0;
}
```

Salida:

```text
1 argumentos
0: C:\Users\zyoan\AppData\Local\Temp\fx_759du3zo\main.exe
```

Con `programa hola 42` los argumentos que llegan a `argv` (sin contar `argv[0]`, el nombre del programa) son `hola` y `42`, siempre como texto (`char*`); aquí no se ve ninguno porque la página se genera ejecutando el programa sin argumentos.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Mezclar avisos con la salida normal | Los avisos van con `std::cerr`, no con `std::cout` |
| Esperar que `std::cout` muestre algo al momento | El terminal a veces retiene la salida: fuerza con `.flush()` o usa `std::endl` (que también vacía el búfer) |
| Pensar que `argv` solo trae los argumentos | `argv[0]` es **siempre** el nombre del programa |
| Dejar un manipulador puesto sin querer | `std::fixed`, `std::setw`... afectan a **todo** lo que se imprima después, hasta que lo cambies |

## Para practicar

Haz los [ejercicios 7.1 de consola](consola.md).
