# 1.3 Tipos de datos

C++ es de **tipado estático**: cada variable tiene un tipo que se conoce al compilar. Con `auto` el compilador lo deduce del valor inicial, pero el tipo sigue existiendo y no cambia después.

## Tipos básicos

| Tipo | Ejemplo | Nota |
|---|---|---|
| `int` | `42` | entero con signo (normalmente 32 bits) |
| `long long` | `42LL` | entero con signo de al menos 64 bits |
| `unsigned int` | `42u` | entero **sin signo** (solo positivos; nunca para restar edades o tamaños que podrían dar negativo) |
| `double` | `3.14` | decimal de 64 bits (el habitual) |
| `float` | `3.14f` | decimal de 32 bits (menos preciso, ocupa menos) |
| `bool` | `true` | solo `true` o `false` (se guarda y muestra como `1`/`0`) |
| `char` | `'A'` | un carácter (1 byte: solo ASCII sin trucos adicionales) |
| `std::string` | `"hola"` | texto que puede crecer (de `<string>`); **úsalo en vez de `char*`** |

!!! warning "Dinero con enteros, no con `double`"
    `double` guarda los decimales en binario y no es exacto: `0.1 + 0.2` da `0.30000000000000004`. Para importes, guarda **céntimos en un entero** o usa una librería de decimales exactos.

## `std::string` frente a `char*`

C hereda las cadenas como arrays de `char` terminados en `\0` (`char*` o `const char*`). C++ añade `std::string`, que gestiona su propia memoria, crece sola y tiene métodos (`.length()`, `.substr()`...). **Usa siempre `std::string`** salvo que tengas una razón concreta para `char*` (interoperar con una librería en C, por ejemplo).

```cpp
#include <iostream>
#include <string>
#include <algorithm>

int main() {
    std::string s = "C++";
    std::cout << s.length() << std::endl;                 // 3 (bytes, no "letras visuales")
    std::string copia = s;
    std::transform(copia.begin(), copia.end(), copia.begin(), ::toupper);
    std::cout << copia << std::endl;                       // C++ (aquí no cambia nada visible, pero funciona con letras)
    std::cout << ("a,b,c" == std::string("a,b,c")) << std::endl;   // compara contenido, no identidad
    std::string saludo = "Hola";
    saludo += ", mundo";                                   // std::string crece con +=
    std::cout << saludo << std::endl;
    return 0;
}
```

Salida:

```text
3
C++
1
Hola, mundo
```

`std::string` no trae `toUpperCase()` como tal: hay que recorrerla con `std::transform` y `::toupper` (o, carácter a carácter, con un bucle). Es uno de los precios de una biblioteca estándar más antigua que la de lenguajes más recientes.

## Colecciones

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <set>
#include <array>

int main() {
    std::vector<int> lista = {3, 1, 2};    // vector: crece dinámicamente
    std::array<int, 3> fijo = {10, 20, 30};   // tamaño fijo, conocido al compilar
    std::set<int> conjunto = {1, 1, 2};        // sin repetidos
    std::map<std::string, int> mapa = {{"a", 1}, {"b", 2}};   // clave -> valor, ordenado por clave

    lista.push_back(4);
    for (int x : lista) std::cout << x << " ";
    std::cout << std::endl;                    // 3 1 2 4
    std::cout << conjunto.size() << std::endl;  // 2
    std::cout << mapa["a"] << std::endl;        // 1
    std::cout << fijo.size() << std::endl;      // 3
    return 0;
}
```

Salida:

```text
3 1 2 4 
2
1
3
```

`std::vector<T>` es la lista que crece, la colección que más se usa. `std::array<T, N>` es su equivalente de tamaño **fijo**, conocido al compilar (más eficiente cuando el tamaño no cambia). `std::map`/`std::set` mantienen sus elementos **ordenados** por clave/valor; si no te importa el orden y quieres más velocidad, existen `std::unordered_map`/`std::unordered_set`.

## Conversiones

```cpp
#include <iostream>
#include <string>

int main() {
    int n = std::stoi("42");               // texto -> entero
    std::cout << n << std::endl;            // 42
    std::cout << (std::to_string(42) + "!") << std::endl;   // 42!
    std::cout << static_cast<int>(3.9) << std::endl;         // 3  (trunca)
    std::cout << static_cast<double>(3) << std::endl;        // 3
    long grande = n;                        // conversión implícita: int cabe en long, no pierde información
    std::cout << grande << std::endl;
    return 0;
}
```

Salida:

```text
42
42!
3
3
42
```

- **Ampliar** (`int` a `long`, `float` a `double`) ocurre **automáticamente**, sin pedirlo, porque no se pierde información.
- **Reducir** (`double` a `int`, `long` a `int`) con `static_cast<T>(valor)`: trunca sin avisar, puede perder información. También se puede escribir `(int) valor` (estilo C), pero `static_cast` deja más claro qué se pretende y el compilador lo comprueba mejor.
- `std::stoi`/`std::stod` lanzan una excepción (`std::invalid_argument`) si el texto no es un número; se controla con `try`/`catch` (unidad 2).

!!! note "Los decimales siempre con punto"
    `std::stod("3,5")` falla: C++ no mira el idioma del equipo salvo que configures explícitamente la *locale*. `3.5` se escribe siempre con punto. Si el usuario escribe con coma, sustitúyela antes de convertir.

## Un valor que puede faltar

C++ no tiene un `null` genérico para cualquier tipo (los punteros sí pueden ser `nullptr`, pero eso es otra cosa: una dirección de memoria). Para "este valor puede no existir", la biblioteca estándar moderna trae `std::optional<T>` (desde C++17): vale `std::nullopt` o contiene un valor, y hay que comprobarlo antes de usarlo (`.has_value()`, o `.value_or(porDefecto)`).
