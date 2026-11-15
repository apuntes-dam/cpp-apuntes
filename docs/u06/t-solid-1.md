# 6.B SOLID: S y O

**SOLID** son cinco principios para que un diseño orientado a objetos siga siendo fácil de cambiar cuando el programa crece.

## S — Responsabilidad única

Una clase debería tener **una sola razón para cambiar**. Si una clase hace dos cosas no relacionadas, un cambio en una puede romper o complicar la otra sin necesidad.

```cpp
#include <iostream>
#include <string>

// Mal: Factura mezcla el cálculo con cómo se presenta
struct FacturaMal {
    double total;
    std::string imprimir() const { return "Total: " + std::to_string(total); }
};

// Mejor: una clase calcula, otra se encarga de presentar
class Factura {
    double total;
public:
    Factura(double total) : total(total) {}
    double getTotal() const { return total; }
};

class ImpresoraFactura {
public:
    static std::string imprimir(const Factura& f) {
        return "Total: " + std::to_string(f.getTotal());
    }
};

int main() {
    Factura f(42.5);
    std::cout << ImpresoraFactura::imprimir(f) << std::endl;
    return 0;
}
```

Salida:

```text
Total: 42.500000
```

Si cambia el **formato** de impresión (a HTML, a PDF...), solo cambia `ImpresoraFactura`; si cambia la **lógica de cálculo** del total, solo cambia `Factura`. Ninguna de las dos razones de cambio obliga a tocar la otra clase.

## O — Abierto/cerrado

Una clase debería poder **extenderse** (añadir comportamiento nuevo) sin **modificar** su código ya probado. `virtual` y las clases abstractas son la herramienta de C++ para esto: el código que usa la base no cambia cuando aparece una hija nueva.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Descuento {
public:
    virtual double aplicar(double precio) const = 0;
    virtual ~Descuento() = default;
};

class SinDescuento : public Descuento {
public:
    double aplicar(double precio) const override { return precio; }
};

class DescuentoPorcentaje : public Descuento {
    double porcentaje;
public:
    DescuentoPorcentaje(double porcentaje) : porcentaje(porcentaje) {}
    double aplicar(double precio) const override { return precio * (1 - porcentaje / 100); }
};

double precioFinal(double precio, const Descuento& d) {
    return d.aplicar(precio);
}

int main() {
    std::cout << precioFinal(100, SinDescuento{}) << std::endl;
    std::cout << precioFinal(100, DescuentoPorcentaje(20)) << std::endl;
    return 0;
}
```

Salida:

```text
100
80
```

`precioFinal` no sabe cuántos tipos de descuento existen ni los conoce por nombre: funciona con **cualquier** `Descuento`. Añadir un tercer tipo de descuento (por ejemplo, uno de cantidad fija) no obliga a tocar ni `precioFinal` ni los descuentos que ya existían: la clase está **abierta** a esa extensión y **cerrada** a modificación.

## Para practicar

Estas ideas se aplican en la [unidad 4](../u04/prueba.md) al diseñar las clases del proyecto.
