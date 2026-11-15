# 5.B Polimorfismo y clases abstractas

## Por qué `virtual` importa

Sin `virtual`, llamar a un método a través de un **puntero o referencia a la base** ejecuta siempre la versión de la base, aunque el objeto real sea de una clase hija. Con `virtual`, se ejecuta la versión real del objeto: esto es el **polimorfismo**.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Figura {
public:
    virtual double area() const = 0;        // puro virtual: sin cuerpo, obliga a cada hija a definirlo
    virtual ~Figura() = default;             // destructor virtual (ver más abajo): obligatorio en una base polimórfica
};

class Cuadrado : public Figura {
    double lado;
public:
    Cuadrado(double lado) : lado(lado) {}
    double area() const override { return lado * lado; }
};

class Circulo : public Figura {
    double radio;
public:
    Circulo(double radio) : radio(radio) {}
    double area() const override { return 3.14159 * radio * radio; }
};

int main() {
    std::vector<std::unique_ptr<Figura>> figuras;
    figuras.push_back(std::make_unique<Cuadrado>(3));
    figuras.push_back(std::make_unique<Circulo>(2));

    for (const auto& f : figuras) {
        std::cout << "área: " << f->area() << std::endl;   // llama a la versión correcta de cada una
    }
    return 0;
}
```

Salida:

```text
área: 9
área: 12.5664
```

`Figura` es una **clase abstracta**: tiene al menos un método **puro virtual** (`= 0`, sin cuerpo), así que no se puede crear un `Figura` directamente, solo sus hijas. El `std::vector<std::unique_ptr<Figura>>` guarda figuras **distintas** detrás del mismo tipo base, y cada `f->area()` ejecuta la versión correcta gracias a `virtual`.

## Dos trampas reales de C++

**El destructor virtual.** Si una clase base tiene métodos `virtual` y se va a borrar un objeto hijo **a través de un puntero a la base** (como `delete` sobre un `Figura*`), la base **necesita** un destructor `virtual` (como el de arriba). Sin él, `delete` solo ejecutaría el destructor de `Figura`, nunca el de la hija: en una clase con recursos propios (memoria, un fichero abierto) eso es una fuga real, y el comportamiento es *indefinido* en el estándar. `std::unique_ptr`/`std::vector` no evitan este problema por sí solos: lo evita tener el destructor `virtual` en la base, siempre.

**El *slicing* (recorte de objetos).** Copiar un objeto hijo **por valor** en una variable del tipo base pierde la parte añadida por la hija: solo se copia el trozo que coincide con la base.

```cpp
#include <iostream>

class Base {
public:
    virtual void saludar() const { std::cout << "Soy Base" << std::endl; }
};

class Hija : public Base {
public:
    void saludar() const override { std::cout << "Soy Hija" << std::endl; }
    int extra = 42;
};

int main() {
    Hija h;
    Base copia = h;          // ¡por valor! Se "recorta" a solo la parte de Base
    copia.saludar();         // imprime "Soy Base": el polimorfismo no funciona aquí
    Base& ref = h;           // por referencia, no hay recorte
    ref.saludar();           // imprime "Soy Hija"
    return 0;
}
```

Salida:

```text
Soy Base
Soy Hija
```

`Base copia = h;` crea un `Base` **nuevo**, copiando solo esa parte; `extra` y el `saludar` de `Hija` desaparecen. Por eso el polimorfismo real necesita punteros o referencias a la base (`Base&`, `Base*`, `std::unique_ptr<Base>`), nunca una copia por valor.

## Para practicar

No hay ejercicios específicos de esta página; las ideas se repasan en la [6.C](../u06/t-solid-2.md) al hablar de sustitución.
