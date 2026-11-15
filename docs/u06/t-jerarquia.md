# 6.A Diseñar una jerarquía

Antes de escribir `class Hija : public Base`, conviene decidir **qué va en cada sitio**.

## Qué va en la base

Solo lo que **de verdad** comparten todas las hijas, y nada más: atributos y métodos comunes, y los métodos `virtual` que cada hija deba poder cambiar. Si una «clase base» solo tiene sentido para una de las hijas, probablemente no es una base, es esa hija disfrazada.

```cpp
#include <iostream>
#include <string>

class Empleado {
protected:
    std::string nombre;
    double salarioBase;

public:
    Empleado(std::string nombre, double salarioBase) : nombre(std::move(nombre)), salarioBase(salarioBase) {}
    virtual double calcularSalario() const { return salarioBase; }
    virtual ~Empleado() = default;
};

class Comercial : public Empleado {
    double comision;
public:
    Comercial(std::string nombre, double salarioBase, double comision) : Empleado(std::move(nombre), salarioBase), comision(comision) {}
    double calcularSalario() const override { return salarioBase + comision; }
};

int main() {
    Comercial c("Ana", 1200, 300);
    std::cout << c.calcularSalario() << std::endl;
    return 0;
}
```

Salida:

```text
1500
```

`nombre` y `salarioBase` son de **todo** empleado, así que van en `Empleado`; `comision` solo tiene sentido para `Comercial`, así que va solo ahí. `calcularSalario` es `virtual` porque cada tipo de empleado lo calcula de forma distinta.

## Responsabilidades del constructor

Cada constructor solo debe inicializar **lo suyo**: la hija llama al constructor de la base para lo heredado, y añade lo propio en su lista de inicialización (como arriba, `Empleado(std::move(nombre), salarioBase)`). Si una hija tuviera que repetir la lógica de inicialización de la base, es una señal de que la base no quedó bien diseñada.

## Para practicar

No hay ejercicio específico de esta página: es la base para pensar las siguientes.
