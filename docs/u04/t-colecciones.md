# 4.D Colecciones y miembros estáticos

## Un `std::vector` de objetos

Igual que un `std::vector<int>` guarda enteros, un `std::vector<Producto>` guarda objetos completos:

```cpp
#include <iostream>
#include <string>
#include <vector>

class Producto {
public:
    std::string nombre;
    double precio;

    Producto(std::string nombre, double precio) : nombre(std::move(nombre)), precio(precio) {}
};

int main() {
    std::vector<Producto> carrito;
    carrito.emplace_back("Pan", 1.20);
    carrito.emplace_back("Leche", 0.95);

    double total = 0;
    for (const auto& p : carrito) {
        std::cout << p.nombre << ": " << p.precio << std::endl;
        total += p.precio;
    }
    std::cout << "Total: " << total << std::endl;
    return 0;
}
```

Salida:

```text
Pan: 1.2
Leche: 0.95
Total: 2.15
```

`emplace_back(...)` construye el `Producto` **directamente dentro** del vector, con esos argumentos, en vez de crear uno aparte y copiarlo (lo que haría `push_back(Producto(...))`). El recorrido `for (const auto& p : carrito)` evita copiar cada elemento.

## Miembros `static`

Un miembro `static` pertenece a la **clase**, no a cada objeto: todos los objetos lo comparten.

```cpp
#include <iostream>

class Contador {
private:
    static int totalCreados;

public:
    Contador() { totalCreados++; }
    static int getTotalCreados() { return totalCreados; }
};

int Contador::totalCreados = 0;   // los static de datos se definen (una vez) fuera de la clase

int main() {
    Contador a, b, c;
    std::cout << "Creados: " << Contador::getTotalCreados() << std::endl;
    return 0;
}
```

Salida:

```text
Creados: 3
```

`totalCreados` existe una sola vez, compartido por `a`, `b` y `c`; cada constructor lo incrementa. Un método `static` (como `getTotalCreados`) se llama sobre la **clase** (`Contador::getTotalCreados()`), no sobre un objeto, y por eso no tiene `this`: solo puede tocar otros miembros `static`.

## Para practicar

Los [ejercicios de robots I](robots-1.md) y [robots II](robots-2.md) practican colecciones de objetos.
