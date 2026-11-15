# 4.C Constructores y enumerados

## Varios constructores

Una clase puede tener **varios constructores** (sobrecarga) o parámetros con **valor por defecto**, para cubrir distintas formas de crear un objeto:

```cpp
#include <iostream>
#include <string>

class Pizza {
public:
    std::string tamano;
    std::string ingrediente;

    Pizza(std::string tamano, std::string ingrediente = "Margarita") : tamano(std::move(tamano)), ingrediente(std::move(ingrediente)) {}

    void describir() const {
        std::cout << tamano << " de " << ingrediente << std::endl;
    }
};

int main() {
    Pizza p1("mediana");
    Pizza p2("grande", "cuatro quesos");
    p1.describir();
    p2.describir();
    return 0;
}
```

Salida:

```text
mediana de Margarita
grande de cuatro quesos
```

Con un parámetro por defecto, `Pizza("mediana")` y `Pizza("grande", "cuatro quesos")` son ambas llamadas válidas al **mismo** constructor. Para construcciones muy distintas entre sí, se declaran constructores **sobrecargados** (mismo nombre, distintos parámetros); el compilador elige el que coincide.

## `enum class`

Un enumerado con **ámbito propio** (a diferencia del `enum` clásico de C, que vuelca sus nombres al espacio exterior y se convierte a `int` sin avisar):

```cpp
#include <iostream>

enum class Tamano { Pequena, Mediana, Grande };

std::string nombreTamano(Tamano t) {
    switch (t) {
        case Tamano::Pequena: return "pequeña";
        case Tamano::Mediana: return "mediana";
        case Tamano::Grande:  return "grande";
    }
    return "?";
}

int main() {
    Tamano t = Tamano::Grande;
    std::cout << nombreTamano(t) << std::endl;
    // t == 2 no compilaría: un enum class no se convierte a int en silencio
    std::cout << (t == Tamano::Grande) << std::endl;
    return 0;
}
```

Salida:

```text
grande
1
```

Para usar un valor hay que escribir `Tamano::Grande` (nunca solo `Grande`), y comparar con otro `Tamano` directamente; para convertir a `int` explícitamente se usa `static_cast<int>(t)`. Es la opción preferida frente al `enum` clásico precisamente porque evita confundir enumerados distintos o compararlos con números por accidente.

## Para practicar

Sigue con los [ejercicios de POO II](poo-2.md).
