# 4.A Clases y objetos

Hasta ahora los datos (variables) y las operaciones (funciones) iban por separado. En la **programación orientada a objetos (POO)** se agrupan en una sola pieza: el **objeto**, una instancia de una **clase** que guarda sus propios datos (**atributos**) y sabe hacer sus propias operaciones (**métodos**).

## Una clase mínima

```cpp
#include <iostream>
#include <string>

class Persona {
public:
    std::string nombre;
    int edad;

    void saludar() const {
        std::cout << "Hola, soy " << nombre << " y tengo " << edad << " años" << std::endl;
    }
};

int main() {
    Persona ana;
    ana.nombre = "Ana";
    ana.edad = 20;
    ana.saludar();
    return 0;
}
```

Salida:

```text
Hola, soy Ana y tengo 20 años
```

`class Persona { ... }` declara el **tipo**; `Persona ana;` crea un **objeto** (una instancia) y `ana.nombre`/`ana.saludar()` acceden a sus miembros con `.`. El `const` al final de `saludar()` promete que el método no modifica el objeto; el compilador lo comprueba.

## Un constructor

Dejar que cualquiera asigne `nombre`/`edad` después de crear el objeto permite dejarlo a medio construir. Un **constructor** obliga a dar los datos necesarios desde el principio:

```cpp
#include <iostream>
#include <string>

class Persona {
public:
    std::string nombre;
    int edad;

    Persona(std::string nombre, int edad) : nombre(std::move(nombre)), edad(edad) {}

    void saludar() const {
        std::cout << "Hola, soy " << nombre << " y tengo " << edad << " años" << std::endl;
    }
};

int main() {
    Persona ana("Ana", 20);
    ana.saludar();
    return 0;
}
```

Salida:

```text
Hola, soy Ana y tengo 20 años
```

`: nombre(std::move(nombre)), edad(edad)` es la **lista de inicialización**: inicializa cada atributo directamente con el valor recibido, antes de que el cuerpo `{}` se ejecute. Es preferible a asignar dentro del cuerpo (`this->nombre = nombre;`), sobre todo cuando el atributo no tiene un valor por defecto razonable. `std::move` evita copiar la cadena que ya nos han pasado por valor.

## Para practicar

Esta idea se practica en los [ejercicios de repaso](repaso.md) y en los dos bloques de [POO I](poo-1.md) y [POO II](poo-2.md).
