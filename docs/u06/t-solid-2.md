# 6.C SOLID: L, I y D

## L — Sustitución de Liskov

Si `Hija` hereda de `Base`, cualquier código que funcione con un `Base` debería seguir funcionando igual de bien si en su lugar se le da una `Hija`, **sin sorpresas**. La [5.B](../u05/t-abstractas.md) ya mostró dos formas reales en las que C++ rompe esto si no tienes cuidado: el *slicing* (al pasar por **valor** en vez de por referencia, la hija deja de comportarse como hija) y olvidar el destructor `virtual` en la base (borrar una hija a través de un puntero a la base sin destructor virtual no la destruye correctamente). Sustituir de verdad significa pasar siempre por referencia o puntero, y declarar `virtual ~Base() = default;` en cualquier base pensada para la herencia.

Hay una trampa más sutil: una hija que **hace menos** de lo que promete la base (por ejemplo, un método que en la base nunca lanza una excepción, pero en la hija sí) también rompe la sustitución, aunque compile sin avisos.

## I — Interfaces pequeñas

Una clase no debería depender de métodos que **no usa**. Igual que en la [5.C](../u05/t-interfaces.md), varias interfaces (clases con solo métodos puros virtuales) pequeñas y concretas son mejores que una grande y genérica:

```cpp
#include <iostream>

// Mal: una interfaz "todo en uno" obliga a implementar lo que no se necesita
class DispositivoMal {
public:
    virtual void imprimir() = 0;
    virtual void escanear() = 0;
};

// Mejor: interfaces separadas, cada clase implementa solo lo que de verdad hace
class Imprimible {
public:
    virtual void imprimir() = 0;
    virtual ~Imprimible() = default;
};

class Escaneable {
public:
    virtual void escanear() = 0;
    virtual ~Escaneable() = default;
};

class ImpresoraSimple : public Imprimible {
public:
    void imprimir() override { std::cout << "Imprimiendo..." << std::endl; }
};

int main() {
    ImpresoraSimple i;
    i.imprimir();
    return 0;
}
```

Salida:

```text
Imprimiendo...
```

`ImpresoraSimple` solo implementa `Imprimible`: no tiene que fingir que sabe escanear (con un método vacío o que lanza una excepción) solo porque la interfaz grande lo exigiera.

## D — Inversión de dependencias

Una clase que necesita avisar, guardar datos o leer de algún sitio no debería depender de **una implementación concreta**, sino de una **abstracción** (una clase con métodos virtuales), que recibe **inyectada** (por el constructor):

```cpp
#include <iostream>
#include <memory>
#include <string>

class Notificador {
public:
    virtual void enviar(const std::string& mensaje) const = 0;
    virtual ~Notificador() = default;
};

class NotificadorConsola : public Notificador {
public:
    void enviar(const std::string& mensaje) const override {
        std::cout << "[consola] " << mensaje << std::endl;
    }
};

class NotificadorSilencioso : public Notificador {
public:
    void enviar(const std::string&) const override { /* no hace nada: útil para pruebas */ }
};

class ServicioPedidos {
    std::unique_ptr<Notificador> notificador;
public:
    ServicioPedidos(std::unique_ptr<Notificador> notificador) : notificador(std::move(notificador)) {}
    void confirmar(const std::string& pedido) {
        notificador->enviar("Pedido confirmado: " + pedido);
    }
};

int main() {
    ServicioPedidos servicio(std::make_unique<NotificadorConsola>());
    servicio.confirmar("#1024");
    return 0;
}
```

Salida:

```text
[consola] Pedido confirmado: #1024
```

`ServicioPedidos` depende de `Notificador` (la abstracción), nunca de `NotificadorConsola` directamente. Se puede cambiar a un notificador por correo, o a uno silencioso para pruebas (como `NotificadorSilencioso`), sin tocar una sola línea de `ServicioPedidos`: es la misma idea de la [6.B](t-solid-1.md), llevada a cómo se **construyen** las clases, no solo a cómo se usan.

## Para practicar

La [unidad 4](../u04/proyecto.md) es donde se junta todo: SRP, OCP, LSP, ISP y DIP a la vez en un proyecto propio.
