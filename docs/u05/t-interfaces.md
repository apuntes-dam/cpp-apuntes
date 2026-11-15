# 5.C Interfaces

C++ no tiene una palabra clave `interface`. El equivalente es una clase **completamente abstracta**: todos sus métodos son puros virtuales, no tiene atributos propios, y solo define un **contrato** que otras clases deben cumplir.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Notificable {
public:
    virtual void notificar(const std::string& mensaje) const = 0;
    virtual ~Notificable() = default;
};

class PorCorreo : public Notificable {
public:
    void notificar(const std::string& mensaje) const override {
        std::cout << "Correo: " << mensaje << std::endl;
    }
};

class PorSms : public Notificable {
public:
    void notificar(const std::string& mensaje) const override {
        std::cout << "SMS: " << mensaje << std::endl;
    }
};

int main() {
    std::vector<std::unique_ptr<Notificable>> canales;
    canales.push_back(std::make_unique<PorCorreo>());
    canales.push_back(std::make_unique<PorSms>());
    for (const auto& c : canales) c->notificar("Pedido enviado");
    return 0;
}
```

Salida:

```text
Correo: Pedido enviado
SMS: Pedido enviado
```

`Notificable` no tiene ningún dato ni ninguna implementación: es puro contrato, exactamente lo que en otros lenguajes es una `interface`. `PorCorreo` y `PorSms` no tienen ninguna relación entre sí más allá de cumplir ese mismo contrato, y una función que reciba un `Notificable&` o `std::unique_ptr<Notificable>` puede trabajar con cualquiera de las dos sin saber cuál es.

## Para practicar

Los [ejercicios de la unidad 4](../u04/robots-1.md) ya usaron una idea parecida con comportamiento como parámetro; esta página es su versión con clases.
