# 5.D Jerarquías cerradas

A veces se sabe de antemano que un conjunto de variantes **no va a crecer**: un semáforo solo tiene tres colores, una operación solo puede salir bien o mal. Para esos casos, C++ ofrece dos herramientas distintas de la herencia abierta.

## `final`: cerrar una clase

```cpp
#include <iostream>

class Base {
public:
    virtual void metodo() const { std::cout << "Base" << std::endl; }
};

class Intermedia final : public Base {   // "final": nadie puede heredar de Intermedia
public:
    void metodo() const override { std::cout << "Intermedia" << std::endl; }
};

// class Nieta : public Intermedia {};   // no compilaría: Intermedia es final

int main() {
    Intermedia i;
    i.metodo();
    return 0;
}
```

Salida:

```text
Intermedia
```

`final` en una clase impide que se herede de ella; en un método (`void metodo() const override final;`), impide que una hija lo vuelva a sobrescribir. Es útil para decir «esta rama de la jerarquía termina aquí», pero sigue siendo herencia: la jerarquía en sí sigue abierta por otros lados.

## `std::variant` + `std::visit`: la alternativa moderna

Para un conjunto **cerrado** de alternativas sin relación entre sí (no todas son "tipos de lo mismo", como `Circulo`/`Cuadrado` sí lo son de `Figura`), muchas veces es más simple no usar herencia en absoluto:

```cpp
#include <iostream>
#include <string>
#include <variant>

struct Exito { int codigo; };
struct Fallo { std::string mensaje; };

using Resultado = std::variant<Exito, Fallo>;

void mostrar(const Resultado& r) {
    std::visit([](const auto& valor) {
        using T = std::decay_t<decltype(valor)>;
        if constexpr (std::is_same_v<T, Exito>) {
            std::cout << "Éxito, código " << valor.codigo << std::endl;
        } else {
            std::cout << "Fallo: " << valor.mensaje << std::endl;
        }
    }, r);
}

int main() {
    mostrar(Exito{200});
    mostrar(Fallo{"no encontrado"});
    return 0;
}
```

Salida:

```text
Éxito, código 200
Fallo: no encontrado
```

`std::variant<Exito, Fallo>` solo puede contener **uno** de esos dos tipos a la vez (nunca ninguno, nunca ambos), sin necesidad de una clase base ni de `virtual`. `std::visit` recibe una función que sabe tratar cada alternativa; `if constexpr` elige en tiempo de compilación qué rama corresponde a cada tipo. Es la jerarquía cerrada «a prueba de bombas»: añadir una tercera alternativa a `Resultado` obliga a tratarla en **todos** los `std::visit` que no cubran ya el caso genérico, así que el compilador avisa de lo que falta actualizar.

## Para practicar

No hay ejercicios específicos de esta página: es la base teórica para diseñar bien la jerarquía de la [unidad 6](../u06/jerarquia.md).
