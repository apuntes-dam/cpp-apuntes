# 4.B Encapsulamiento

**Encapsular** es ocultar cómo está hecho un objeto por dentro y ofrecer solo las operaciones necesarias por fuera. En C++, los miembros de una `class` son **privados por defecto**: hay que marcarlos `public` a propósito.

## Atributos privados y validación

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

class CuentaBancaria {
private:
    std::string titular;
    double saldo;

public:
    CuentaBancaria(std::string titular, double saldoInicial) : titular(std::move(titular)), saldo(0) {
        if (saldoInicial < 0) throw std::invalid_argument("el saldo inicial no puede ser negativo");
        saldo = saldoInicial;
    }

    double getSaldo() const { return saldo; }

    void ingresar(double cantidad) {
        if (cantidad <= 0) throw std::invalid_argument("la cantidad a ingresar debe ser positiva");
        saldo += cantidad;
    }

    void retirar(double cantidad) {
        if (cantidad > saldo) throw std::runtime_error("fondos insuficientes");
        saldo -= cantidad;
    }
};

int main() {
    CuentaBancaria cuenta("Ana", 100);
    cuenta.ingresar(50);
    cuenta.retirar(30);
    std::cout << "Saldo: " << cuenta.getSaldo() << std::endl;
    try {
        cuenta.retirar(1000);
    } catch (const std::runtime_error& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }
    return 0;
}
```

Salida:

```text
Saldo: 120
Error: fondos insuficientes
```

`saldo` solo se puede cambiar a través de `ingresar`/`retirar`, que **validan** antes de modificar (ver la unidad 2 para `try`/`catch`). Desde fuera de la clase, `cuenta.saldo` ni siquiera compila: es `private`.

## Igualdad: `operator==` y `operator<=>`

Comparar dos objetos con `==` no compila por defecto: hay que decir qué significa que sean «iguales». Se puede escribir a mano, o con C++20 pedirle al compilador que lo genere:

```cpp
#include <iostream>

struct Punto {
    int x, y;
    auto operator<=>(const Punto&) const = default;   // genera ==, !=, <, >, <=, >= comparando x y luego y
};

int main() {
    Punto a{1, 2}, b{1, 2}, c{3, 4};
    std::cout << std::boolalpha << (a == b) << " " << (a == c) << " " << (c > a) << std::endl;
    return 0;
}
```

Salida:

```text
true false true
```

`<=>` (el operador de «comparación de tres vías», *spaceship*) compara miembro a miembro en el orden en que están declarados; con `= default` el compilador lo escribe por ti y, de regalo, también `==`. Para una clase con lógica propia de igualdad (por ejemplo, ignorar mayúsculas en un nombre), se sobrecarga `operator==` a mano en vez de usar `<=>`.

## Para practicar

Los [ejercicios de POO I](poo-1.md) practican validación y encapsulamiento.
