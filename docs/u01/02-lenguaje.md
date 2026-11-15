# 1.2 Práctica con el lenguaje

## Estructura de un programa C++

Todo programa empieza en la función `main`, que devuelve un `int` (el **código de salida**: `0` significa que todo fue bien). Las funciones se declaran con su tipo de retorno delante:

```cpp
#include <iostream>

int cuadrado(int n) {
    return n * n;
}

int main() {
    std::cout << "Hola, C++" << std::endl;
    std::cout << cuadrado(5) << std::endl;   // 25
    return 0;
}
```

Salida:

```text
Hola, C++
25
```

Reglas básicas: cada instrucción termina en `;`, los bloques van entre `{ }` y C++ **distingue mayúsculas de minúsculas**. `#include <iostream>` incorpora la entrada/salida estándar; `std::` indica que `cout`/`cin`/`endl` vienen del **espacio de nombres** `std` (se puede escribir `using namespace std;` para no repetirlo, pero en archivos grandes es mejor no hacerlo: evita choques de nombres).

Las funciones y variables suelen escribirse en `minusculas_con_guion` o `camelCase` (ambos son habituales, a diferencia de otros lenguajes no hay una única convención oficial), y los tipos propios en `PascalCase`.

## Variables

```cpp
#include <iostream>
#include <string>

int main() {
    std::string nombre = "Ana";   // tipo explícito siempre, o `auto` para que el compilador lo deduzca
    int edad = 20;
    edad = 21;                     // se puede cambiar: no hay nada parecido a "inmutable por defecto"
    double altura = 1.68;
    double peso;                   // declarada, sin valor: ¡cuidado! su contenido es indeterminado hasta que se asigna
    peso = 62.5;
    std::cout << nombre << " tiene " << edad << " años, mide " << altura << " y pesa " << peso << std::endl;
    return 0;
}
```

Salida:

```text
Ana tiene 21 años, mide 1.68 y pesa 62.5
```

A diferencia de Rust o Kotlin, en C++ **todas las variables son modificables por defecto**: para impedir que cambien hace falta `const` (ver más abajo). Una variable declarada sin inicializar (`double peso;`) tiene un valor **indeterminado** hasta que se le asigna uno: usarla antes es un error que el compilador no siempre detecta.

!!! tip "`auto`: que el compilador deduzca el tipo"
    `auto nombre = "Ana";` funciona, pero ahí `"Ana"` es literalmente `const char*` (una cadena de estilo C), no `std::string`. Para que `auto` deduzca `std::string`, escribe `auto nombre = std::string("Ana");` o, más simple, declara el tipo explícito como arriba. `auto` es más útil cuando el tipo es largo o evidente por el contexto.

## Constantes

| Forma | Cuándo se fija | Ejemplo |
|---|---|---|
| `const` | Al declararla; puede venir de un cálculo en tiempo de ejecución | `const double iva = 0.21;` |
| `constexpr` | Obligatoriamente en compilación | `constexpr double IVA = 0.21;` |

Una variable `const` no se puede reasignar después: `const int x = 5; x = 6;` no compila. `constexpr` es más estricto: exige que el valor se conozca **al compilar**, y permite usarla donde C++ exige una constante de compilación (el tamaño de un array de tamaño fijo, por ejemplo).

## Literales

`42` (`int`), `42u` (`unsigned int`), `42L` (`long`), `3.14` (`double`), `3.14f` (`float`), `'a'` (`char`), `"hola"` (`const char*`, una cadena de estilo C) y `true`/`false` (`bool`). Los miles se pueden separar con un apóstrofo: `1'000'000`. Hexadecimal `0xff`, octal `010`, binario `0b1010`.

## Operadores

```cpp
#include <iostream>

int main() {
    std::cout << (7 + 2) << std::endl;          // 9
    std::cout << (7 / 2) << std::endl;          // 3    (entero / entero = división entera)
    std::cout << (7.0 / 2.0) << std::endl;      // 3.5  (con al menos un operando decimal)
    std::cout << (7 % 2) << std::endl;          // 1
    std::cout << (3 > 2 && 2 > 1) << std::endl; // 1    (bool se muestra como 0/1 salvo con boolalpha)
    int x = 5;
    x += 2;    // 7
    x++;       // 8  (sí existe, a diferencia de Rust o Python)
    std::cout << x << std::endl;
    std::cout << (x > 5 ? "grande" : "pequeño") << std::endl;   // operador ternario: si/sino como expresión
    return 0;
}
```

Salida:

```text
9
3
3.5
1
1
8
grande
```

Grupos: aritméticos (`+ - * / %`), relacionales (`== != < > <= >=`), lógicos (`&& || !`) y de asignación (`= += -= *= /=`), más `++`/`--` (incremento y decremento) y el ternario `cond ? a : b`. Al mezclar un `int` con un `double` en una operación, C++ **convierte automáticamente** el `int` a `double` (al revés de Rust, que exige convertir a mano): `ancho / 2.0` ya da una división decimal sin necesidad de `static_cast`.

!!! tip "Mostrar `bool` como texto"
    `std::cout << valor;` muestra `1` o `0`. Para ver `true`/`false`, usa `std::cout << std::boolalpha << valor;` (afecta a todo lo que se imprima después, hasta `std::noboolalpha`).

## Entrada y salida

```cpp
#include <iostream>
#include <string>

int main() {
    std::cout << "¿Cómo te llamas? ";
    std::string nombre;
    std::getline(std::cin, nombre);           // lee una línea completa (con espacios)
    std::cout << "Hola, " << nombre << std::endl;
    std::cout << "Tienes " << (20 + 1) << " años" << std::endl;   // las expresiones se evalúan antes de imprimir
    return 0;
}
```

Salida:

```text
¿Cómo te llamas? Hola, Ana
Tienes 21 años
```

`std::cin >> variable` lee un dato **separado por espacios** (un número, o una palabra sin espacios); `std::getline(std::cin, texto)` lee la **línea completa**, incluidos los espacios. No se pueden mezclar sin cuidado: `std::cin >> n` deja el salto de línea sin consumir en el búfer, y un `getline` justo después leería una línea vacía.

!!! note "Leer un número con `getline`"
    Si quieres controlar errores de entrada (ver unidad 2), suele ser más cómodo leer siempre con `getline` y convertir después con `std::stoi`/`std::stod`, que lanzan una excepción si el texto no es un número.

## Comentarios

```cpp
// Comentario de una línea
/* Comentario
   de varias líneas */
/// Comentario de documentación (convención de Doxygen, no es parte del lenguaje)
```
