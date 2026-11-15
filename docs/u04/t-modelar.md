# 4.E Del problema al programa

## Qué se convierte en clase

No todo tiene que ser una clase. Antes de crear una, pregúntate: **¿este concepto tiene datos propios Y comportamiento propio que tenga sentido agrupar?** Un `Pedido` con su lista de líneas y su método `total()` sí; una simple pareja de números casi siempre es mejor como `std::pair` o `struct` sin métodos.

## Comportamiento como parámetro

No todo el comportamiento tiene que vivir *dentro* de la clase: a veces conviene recibirlo **desde fuera**, como una función:

```cpp
#include <iostream>
#include <functional>
#include <string>
#include <vector>

class Lista {
public:
    std::vector<int> valores;

    int contarSi(const std::function<bool(int)>& condicion) const {
        int total = 0;
        for (int v : valores) if (condicion(v)) total++;
        return total;
    }
};

int main() {
    Lista l{{1, 2, 3, 4, 5, 6}};
    int pares = l.contarSi([](int n) { return n % 2 == 0; });
    std::cout << "Pares: " << pares << std::endl;
    return 0;
}
```

Salida:

```text
Pares: 3
```

`contarSi` no sabe nada de «par» o «impar»: recibe la **condición** como un `std::function<bool(int)>` (cualquier cosa llamable con un `int` que devuelva `bool`), y aquí se le pasa una lambda `[](int n) { return n % 2 == 0; }`. Es la misma idea que un criterio de ordenación o un filtro: en vez de escribir una clase distinta por cada condición, se pasa la condición como dato.

## Para practicar

Esta forma de pensar se pone a prueba en la [prueba de la unidad](prueba.md) y el [proyecto final](proyecto.md).
