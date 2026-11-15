# 3.1 Listas y tuplas

Una **lista** guarda varios valores **en orden** y permite consultar por posición. En C++ hay varias formas:

| Tipo | Tamaño | Ejemplo |
|---|---|---|
| **Array** `std::array<T, N>` | **Fijo**, se conoce al compilar | `std::array<int, 3> dias = {1, 2, 3};` |
| **Vector** `std::vector<T>` | **Crece y se encoge** | `std::vector<int> v = {1, 2, 3};` |
| **Array de C** `T[N]` | Fijo, sin métodos | `int dias[3] = {1, 2, 3};` (evítalo salvo que algo lo exija) |

Usa **`std::vector<T>`** casi siempre.

## Operaciones básicas

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numeros = {5, 3, 8};
    numeros.push_back(1);                              // añade al final
    numeros.insert(numeros.begin() + 1, 9);             // inserta en la posición 1
    for (int n : numeros) std::cout << n << " ";
    std::cout << std::endl;                              // 5 9 3 8 1

    std::cout << numeros[0] << " " << numeros.size() << std::endl;
    std::cout << numeros.front() << " " << numeros.back() << std::endl;

    numeros.erase(numeros.begin() + 1);                 // quita por posición
    std::cout << numeros.back() << std::endl;
    numeros.pop_back();                                 // quita el último

    std::cout << (std::find(numeros.begin(), numeros.end(), 3) != numeros.end()) << std::endl;
    std::sort(numeros.begin(), numeros.end());
    for (int n : numeros) std::cout << n << " ";
    std::cout << std::endl;
    std::reverse(numeros.begin(), numeros.end());
    for (int n : numeros) std::cout << n << " ";
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
5 9 3 8 1 
5 5
5 1
1
1
3 5 8 
8 5 3 
```

`numeros[10]` fuera de rango es **comportamiento indefinido** (no avisa); `numeros.at(10)` sí lanza `std::out_of_range`. No hace falta declarar nada especial para poder añadir: basta con que el `vector` no sea `const`.

## Recorrer

```cpp
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> asignaturas = {"Matemáticas", "Física", "Química"};

    for (const auto& a : asignaturas) {                // const&: solo mira, sin copiar
        std::cout << "Yo estudio " << a << std::endl;
    }
    for (size_t i = 0; i < asignaturas.size(); ++i) {   // con la posición
        std::cout << i << ": " << asignaturas[i] << std::endl;
    }

    std::vector<int> notas = {5, 7, 9};
    for (auto& n : notas) {                             // &: puede cambiar cada elemento
        n += 1;
    }
    for (int n : notas) std::cout << n << " ";
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
Yo estudio Matemáticas
Yo estudio Física
Yo estudio Química
0: Matemáticas
1: Física
2: Química
6 8 10 
```

`for (auto a : v)` (sin `&`) **copia** cada elemento; para texto u objetos grandes, usa `const auto&` si solo lees, o `auto&` si vas a modificar.

## Copias, trozos y listas anidadas

Un `std::vector` **se copia** al asignarlo (`copia = original;` ya es una copia independiente, a diferencia de un puntero): no hace falta un método `.clone()` aparte.

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> original = {1, 2, 3, 4, 5};

    std::vector<int> copia = original;                  // copia independiente
    std::vector<int> trozo(original.begin() + 1, original.begin() + 3);
    for (int n : copia) std::cout << n << " ";
    std::cout << "| ";
    for (int n : trozo) std::cout << n << " ";
    std::cout << std::endl;

    // matriz: vector de vectores
    std::vector<std::vector<int>> matriz(2, std::vector<int>(3, 0));   // 2 filas, 3 columnas
    matriz[1][2] = 7;
    for (const auto& fila : matriz) {
        for (int x : fila) std::cout << x << " ";
        std::cout << "| ";
    }
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
1 2 3 4 5 | 2 3 
0 0 0 | 0 0 7 | 
```

## Tuplas

Una **tupla** agrupa valores de **tipos distintos**, de tamaño fijo. `std::pair` es para dos valores; `std::tuple`, para cualquier cantidad. Se accede con `std::get<i>` o, desde C++17, **desestructurando** con *structured bindings*:

```cpp
#include <iostream>
#include <tuple>
#include <string>
#include <vector>

std::tuple<int, int> min_max(const std::vector<int>& valores) {
    int menor = valores[0], mayor = valores[0];
    for (int v : valores) {
        if (v < menor) menor = v;
        if (v > mayor) mayor = v;
    }
    return {menor, mayor};             // una función puede «devolver varios valores»
}

int main() {
    std::tuple<std::string, int, double> persona = {"Ana", 30, 1.68};
    std::cout << std::get<0>(persona) << " " << std::get<1>(persona) << " " << std::get<2>(persona) << std::endl;

    auto [nombre, edad, altura] = persona;    // desestructurar
    std::cout << nombre << " " << edad << std::endl;

    auto [menor, mayor] = min_max({50, 75, 46, 22, 80, 65, 8});
    std::cout << "Menor: " << menor << ", mayor: " << mayor << std::endl;
    return 0;
}
```

Salida:

```text
Ana 30 1.68
Ana 30
Menor: 8, mayor: 80
```

## Transformar con `<algorithm>`

C++ no encadena `map`/`filter` tan directamente como Rust o Python (las *ranges* de C++20 se acercan, pero con una sintaxis más densa); lo habitual es un bucle, o las funciones de `<algorithm>` con un **destino** ya reservado:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>

int main() {
    std::vector<int> numeros = {1, 2, 3, 4, 5, 6};

    std::vector<int> pares;
    std::copy_if(numeros.begin(), numeros.end(), std::back_inserter(pares), [](int n) { return n % 2 == 0; });

    std::vector<int> cuadrados(numeros.size());
    std::transform(numeros.begin(), numeros.end(), cuadrados.begin(), [](int n) { return n * n; });

    int suma = std::accumulate(numeros.begin(), numeros.end(), 0);
    for (int p : pares) std::cout << p << " ";
    std::cout << "| ";
    for (int c : cuadrados) std::cout << c << " ";
    std::cout << "| " << suma << std::endl;

    std::cout << *std::min_element(numeros.begin(), numeros.end()) << " ";
    std::cout << *std::max_element(numeros.begin(), numeros.end()) << std::endl;
    std::cout << std::any_of(numeros.begin(), numeros.end(), [](int n) { return n > 5; }) << std::endl;

    // producto escalar de dos vectores
    std::vector<int> a = {1, 2, 3}, b = {-1, 0, 2};
    int escalar = std::inner_product(a.begin(), a.end(), b.begin(), 0);
    std::cout << escalar << std::endl;
    return 0;
}
```

Salida:

```text
2 4 6 | 1 4 9 16 25 36 | 21
1 6
1
5
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `v[10]` en un vector de 3 elementos | Comportamiento indefinido, no avisa: usa `v.at(10)` si el índice puede estar mal |
| `v.erase`/`v.insert` mientras recorres con un `for` por rango | Invalida los iteradores; usa índices o un segundo vector |
| Copiar un vector grande sin querer en un `for (auto x : v)` | Usa `const auto&` si solo lees |
| Olvidar reservar tamaño antes de `std::transform` | El destino necesita ya el tamaño correcto (o usa `std::back_inserter`) |
| Confundir `.size()` (`size_t`, sin signo) con un `int` en una resta | `v.size() - 10` con una lista de 3 elementos **no da -7**: da un número enorme (sin signo) |

## Para practicar

Haz los [ejercicios 3.1 de listas](listas.md).
