# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que resuelve un problema, y el **algoritmo** son los pasos, en orden, que van de los datos de entrada al resultado.

!!! info "La teoría general está aparte"
    Qué es un algoritmo, el ciclo de desarrollo y el **pseudocódigo** son iguales en todos los lenguajes, así que están en una sola página: [Fundamentos: algoritmos y pseudocódigo](https://apuntes-dam.github.io/apuntes-lenguajes/fundamentos/).

## Del pseudocódigo a C++

El algoritmo del área de un rectángulo, en pseudocódigo:

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR "Área:", area
FIN
```

y su traducción a C++:

```cpp
#include <iostream>

int main() {
    double base, altura;
    std::cout << "Base: ";
    std::cin >> base;
    std::cout << "Altura: ";
    std::cin >> altura;

    double area = base * altura;
    std::cout << "Area: " << area << std::endl;
    return 0;
}
```

Salida:

```text
Base: Altura: Area: 12
```

| Pseudocódigo | C++ |
|---|---|
| `LEER x` | `std::cin >> x;` |
| `x <- expresión` | `x = expresión;` (o `tipo x = expresión;` si es nueva) |
| `ESCRIBIR x` | `std::cout << x;` (y `std::endl` o `"\n"` para el salto de línea) |
| `SI ... SINO ... FIN SI` | `if` / `else` (ver [1.2](02-lenguaje.md)) |
| `PARA` / `MIENTRAS` | `for` / `while` (ver [1.6](06-bucles.md)) |

!!! warning "Errores típicos en C++"
    Olvidar el `;` al final de cada instrucción, olvidar `#include <iostream>` (sin él, `std::cout`/`std::cin` no existen), confundir `=` (asignar) con `==` (comparar) y mezclar `std::cin >>` con `std::getline` sin darse cuenta de que `>>` deja el salto de línea sin leer en el búfer. El compilador de C++ es menos claro que el de otros lenguajes modernos: un error pequeño a veces produce un mensaje largo y confuso; **lee la primera línea del error primero**, suele ser la más útil.
