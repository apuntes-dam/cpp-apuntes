# 5.A Herencia

**Heredar** permite que una clase (la **hija**) reutilice los atributos y métodos de otra (la **base**), y añada o cambie lo que necesite.

```cpp
#include <iostream>
#include <string>

class Animal {
protected:
    std::string nombre;

public:
    Animal(std::string nombre) : nombre(std::move(nombre)) {}

    void dormir() const {
        std::cout << nombre << " está durmiendo" << std::endl;
    }
};

class Perro : public Animal {
public:
    Perro(std::string nombre) : Animal(std::move(nombre)) {}

    void ladrar() const {
        std::cout << nombre << " dice: ¡Guau!" << std::endl;
    }
};

int main() {
    Perro rex("Rex");
    rex.dormir();     // heredado de Animal
    rex.ladrar();     // propio de Perro
    return 0;
}
```

Salida:

```text
Rex está durmiendo
Rex dice: ¡Guau!
```

`class Perro : public Animal` hace que `Perro` tenga todo lo de `Animal` más lo suyo. El constructor de `Perro` llama **explícitamente** al de `Animal` en su lista de inicialización (`: Animal(std::move(nombre))`); si no lo hiciera, C++ llamaría al constructor **sin parámetros** de `Animal`, y si este no existe, no compila. `nombre` es `protected` (no `private`) precisamente para que `Perro` pueda usarlo directamente.

## Sobrescribir un método

```cpp
#include <iostream>
#include <string>

class Animal {
protected:
    std::string nombre;

public:
    Animal(std::string nombre) : nombre(std::move(nombre)) {}
    virtual void hacerSonido() const {
        std::cout << nombre << " hace un sonido" << std::endl;
    }
};

class Gato : public Animal {
public:
    Gato(std::string nombre) : Animal(std::move(nombre)) {}
    void hacerSonido() const override {
        std::cout << nombre << " dice: ¡Miau!" << std::endl;
    }
};

int main() {
    Gato michi("Michi");
    michi.hacerSonido();
    return 0;
}
```

Salida:

```text
Michi dice: ¡Miau!
```

`virtual` en la base marca que el método **puede** sobrescribirse; `override` en la hija comprueba, al compilar, que realmente estás sobrescribiendo algo que existe con esa misma firma (sin `override`, un error de tipeo crearía silenciosamente un método nuevo en vez de sobrescribir). La 5.B explica por qué `virtual` importa de verdad cuando se trabaja a través de la clase base.

## Para practicar

Los [ejercicios de herencia](herencia.md) ponen esto en práctica.
