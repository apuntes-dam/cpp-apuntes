# 3.3 Conjuntos

Un **conjunto** guarda valores **sin repetir** y responde rápido a «¿está este valor?». Como los mapas, hay dos: **`std::set<T>`** (ordenado) y **`std::unordered_set<T>`** (sin orden, más rápido).

## Operaciones

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> vistos;
    std::cout << vistos.insert(3).second << std::endl;   // true: era nuevo
    std::cout << vistos.insert(3).second << std::endl;   // false: ya estaba
    vistos.insert(1);
    for (int v : vistos) std::cout << v << " ";
    std::cout << "(" << vistos.size() << ")" << std::endl;
    std::cout << vistos.count(1) << std::endl;
    vistos.erase(3);
    for (int v : vistos) std::cout << v << " ";
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
1
0
1 3 (2)
1
1 
```

## Unión, intersección y diferencia

`<algorithm>` trae `std::set_union`, `std::set_intersection`, `std::set_difference` y `std::set_symmetric_difference` — **necesitan rangos ya ordenados** (por eso se usan con `std::set`, no con `std::unordered_set`) y un destino donde escribir el resultado:

```cpp
#include <iostream>
#include <set>
#include <vector>
#include <algorithm>
#include <iterator>

int main() {
    std::set<int> a = {1, 2, 3, 4, 5, 6};
    std::set<int> b = {4, 5, 6, 7, 8, 9};

    std::vector<int> union_, interseccion, diferencia, dif_simetrica;
    std::set_union(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(union_));
    std::set_intersection(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(interseccion));
    std::set_difference(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(diferencia));
    std::set_symmetric_difference(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(dif_simetrica));

    auto mostrar = [](const std::string& etiqueta, const std::vector<int>& v) {
        std::cout << etiqueta << ": ";
        for (int x : v) std::cout << x << " ";
        std::cout << std::endl;
    };
    mostrar("unión       ", union_);
    mostrar("intersección", interseccion);
    mostrar("diferencia  ", diferencia);
    mostrar("simétrica   ", dif_simetrica);
    std::cout << std::includes(b.begin(), b.end(), a.begin(), a.end()) << std::endl;   // ¿a es subconjunto de b?
    return 0;
}
```

Salida:

```text
unión       : 1 2 3 4 5 6 7 8 9 
intersección: 4 5 6 
diferencia  : 1 2 3 
simétrica   : 1 2 3 7 8 9 
0
```

## Quitar repetidos

Construir un `std::set` a partir de una lista es la forma más corta de quitar repetidos (pero pierde el orden de aparición, porque un `set` ordena por valor):

```cpp
#include <iostream>
#include <set>
#include <unordered_set>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> compras = {"Nuria", "Jorge", "Nuria", "Eva", "Jorge", "Nuria"};

    std::set<std::string> unicos(compras.begin(), compras.end());
    std::cout << "(" << unicos.size() << " distintos) ";
    for (const auto& n : unicos) std::cout << n << " ";
    std::cout << std::endl;

    // conservando el orden de la primera aparición
    std::unordered_set<std::string> vistos;
    std::vector<std::string> sin_repetir;
    for (const auto& n : compras) {
        if (vistos.insert(n).second) sin_repetir.push_back(n);
    }
    for (const auto& n : sin_repetir) std::cout << n << " ";
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
(3 distintos) Eva Jorge Nuria 
Nuria Jorge Eva 
```

## ¿Vector o conjunto?

| Necesitas... | Usa |
|---|---|
| Conservar el **orden** y los **repetidos** | Vector |
| **Quitar repetidos** o comprobar pertenencia rápido | Conjunto |
| Unión, intersección y diferencia | Conjunto (`std::set`, para usar `<algorithm>`) |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `std::set_union`/`intersection`/... con un `unordered_set` | Necesitan rangos **ordenados**: usa `std::set` |
| Esperar que el resultado de `set_union`... sea un `std::set` | Son algoritmos genéricos: escriben en el **destino** que les pases (normalmente un `std::vector`) |
| Esperar orden de aparición en un `std::set` | Ordena por **valor**, no por cuándo se insertó: si te importa el orden original, usa un `unordered_set` auxiliar como aquí arriba |
| Guardar `double` en un conjunto | Compila, pero las comparaciones de punto flotante son delicadas |

## Para practicar

Haz los [ejercicios 3.3 de conjuntos](conjuntos.md).
