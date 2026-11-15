# 1.4 Pruebas

Una **prueba** comprueba automáticamente que una función devuelve lo esperado. C++ **no trae un framework de pruebas incorporado** (al contrario que Rust con `#[test]` o C# con xUnit): en un proyecto real se añade una librería como [Catch2](https://github.com/catchorg/Catch2) o [GoogleTest](https://github.com/google/googletest).

Para practicar aquí, sin añadir ninguna dependencia, basta con `assert()` (de `<cassert>`): comprueba una condición y, si es falsa, **detiene el programa** con un mensaje que indica el archivo y la línea.

## La función y sus comprobaciones

```cpp
#include <cassert>
#include <iostream>

int suma(int a, int b) {
    return a + b;
}

double dividir(int a, int b) {
    assert(b != 0 && "b no puede ser 0");
    return static_cast<double>(a) / b;
}

int main() {
    assert(suma(2, 3) == 5);
    assert(suma(0, 0) == 0);
    assert(suma(-1, 1) == 0);
    std::cout << "suma: todo correcto" << std::endl;

    assert(dividir(10, 2) == 5.0);
    std::cout << "dividir: todo correcto" << std::endl;
    return 0;
}
```

Salida:

```text
suma: todo correcto
dividir: todo correcto
```

`assert(condición)` no hace nada si la condición es verdadera; si es falsa, imprime un mensaje por el error estándar y aborta. `assert(condición && "mensaje")` es un truco habitual para añadir una explicación: como un `const char*` no vacío siempre es verdadero, el `&&` no cambia el resultado, pero el mensaje aparece en el aviso de fallo.

## Qué pasa cuando un `assert` falla

```cpp
#include <cassert>
#include <iostream>

int suma(int a, int b) {
    return a + b;
}

int main() {
    assert(suma(2, 3) == 5);
    std::cout << "primera comprobacion: OK" << std::endl;
    assert(suma(2, 2) == 99);    // a propósito, falla
    std::cout << "esto no llega a imprimirse" << std::endl;
    return 0;
}
```

**Al ejecutar:**

```text
primera comprobacion: OK
Assertion failed: suma(2, 2) == 99, file main.cpp, line 11
```

El programa se detiene **en la primera comprobación falsa**: las instrucciones de después no se ejecutan (por eso no se ve el segundo mensaje). `assert` es una herramienta de **depuración**: en una compilación "release" con `NDEBUG` definido, los `assert` se desactivan por completo (ni siquiera se evalúa la condición), así que nunca uses `assert` para validar datos del usuario — para eso están las excepciones (unidad 2).

!!! tip "Patrón AAA"
    **A**rrange (preparar los datos), **A**ct (llamar a la función), **A**ssert (comprobar el resultado) — las tres líneas de cada bloque de arriba siguen ese orden.

!!! note "Para un proyecto real"
    Catch2 y GoogleTest añaden lo que `assert` no tiene: pruebas con nombre, que siguen ejecutándose aunque una falle, un resumen final y una sintaxis más expresiva (`REQUIRE(...)`, `EXPECT_EQ(...)`). Se instalan con un gestor de paquetes (vcpkg, Conan) o como una librería header-only.
