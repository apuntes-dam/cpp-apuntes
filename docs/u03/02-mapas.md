# 3.2 Mapas

Un **mapa** guarda **pares clave → valor** y permite encontrar un valor por su clave. En C++ hay dos:

| Tipo | Orden | Cuándo |
|---|---|---|
| **`std::map<K, V>`** | **Ordenado** por clave | Cuando quieres recorrerlo en orden, o claves comparables de forma natural |
| **`std::unordered_map<K, V>`** | **Ninguno** | El más rápido, cuando el orden no importa |

## Crear, consultar, modificar y borrar

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, std::string> divisas;
    divisas["Euro"] = "€";
    divisas["Dollar"] = "$";
    divisas["Yen"] = "¥";

    auto it = divisas.find("Yen");
    std::cout << (it != divisas.end() ? it->second : "no está") << std::endl;
    std::cout << (divisas.find("Peso") != divisas.end()) << std::endl;
    std::cout << divisas["Euro"] << std::endl;              // con [] crea la clave si no existe (¡cuidado!)
    std::cout << divisas.count("Yen") << std::endl;          // 1 si existe, 0 si no
    std::cout << divisas.size() << std::endl;

    divisas["Euro"] = "EUR";                                 // asignar una clave existente la SOBRESCRIBE
    std::cout << divisas["Euro"] << std::endl;

    divisas.erase("Dollar");
    std::cout << divisas.size() << std::endl;

    if (divisas.find("Peso") == divisas.end()) {
        std::cout << "La divisa Peso no está en el diccionario" << std::endl;
    }
    return 0;
}
```

Salida:

```text
¥
0
€
1
3
EUR
2
La divisa Peso no está en el diccionario
```

!!! warning "`mapa["clave"]` CREA la clave si no existe"
    A diferencia de `.at("clave")` (que lanza `std::out_of_range` si falta), el operador `[]` sobre un `map` que **no es `const`** **inserta** la clave con un valor por defecto (una cadena vacía, un `0`...) con solo **leerla**. Para consultar sin crear nada, usa `.find()` o `.count()`.

## Recorrer

`std::map` ya sale ordenado por clave; `std::unordered_map`, no:

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> creditos = {{"Matemáticas", 6}, {"Física", 4}, {"Química", 5}};

    for (const auto& [nombre, cred] : creditos) {     // ya en orden alfabético por clave
        std::cout << nombre << " tiene " << cred << " créditos" << std::endl;
    }

    int total = 0;
    for (const auto& [nombre, cred] : creditos) total += cred;
    std::cout << "Total: " << total << std::endl;
    return 0;
}
```

Salida:

```text
Física tiene 4 créditos
Matemáticas tiene 6 créditos
Química tiene 5 créditos
Total: 15
```

## Contar con un mapa

```cpp
#include <iostream>
#include <map>
#include <sstream>
#include <string>

int main() {
    std::string frase = "la casa de la pradera es la casa";
    std::map<std::string, int> frecuencia;

    std::stringstream flujo(frase);
    std::string palabra;
    while (flujo >> palabra) {
        frecuencia[palabra]++;    // [] crea la clave con 0 la primera vez, y luego se incrementa
    }
    for (const auto& [p, n] : frecuencia) std::cout << p << ":" << n << " ";
    std::cout << std::endl;

    // contar letras
    std::map<char, int> letras;
    for (char c : std::string("banana")) letras[c]++;
    for (const auto& [c, n] : letras) std::cout << c << ":" << n << " ";
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
casa:2 de:1 es:1 la:3 pradera:1 
a:3 b:1 n:2 
```

## Claves, valores y mapas de mapas

```cpp
#include <iostream>
#include <map>
#include <string>
#include <vector>

int main() {
    // cada clave guarda una lista
    std::map<char, std::vector<std::string>> por_inicial;
    for (const std::string& nombre : {"Ana", "Luis", "Alba", "Eva", "Laura"}) {
        por_inicial[nombre[0]].push_back(nombre);
    }
    for (const auto& [inicial, nombres] : por_inicial) {
        std::cout << inicial << ": ";
        for (size_t i = 0; i < nombres.size(); ++i) std::cout << nombres[i] << (i + 1 < nombres.size() ? ", " : "");
        std::cout << std::endl;
    }

    // un mapa dentro de otro
    std::map<std::string, std::map<std::string, std::string>> clientes;
    clientes["01234567L"]["nombre"] = "Luis González";
    clientes["01234567L"]["preferente"] = "true";
    std::cout << clientes["01234567L"]["nombre"] << std::endl;
    return 0;
}
```

Salida:

```text
A: Ana, Alba
E: Eva
L: Luis, Laura
Luis González
```

## ¿Mapa, vector o conjunto?

| Necesitas... | Usa |
|---|---|
| Buscar por **clave** | Mapa |
| Mantener el **orden** de inserción y repetir valores | Vector |
| Saber solo **si algo está** o quitar repetidos | Conjunto ([3.3](03-conjuntos.md)) |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `mapa["x"]` para **consultar** una clave que puede no existir | Crea la clave con un valor por defecto: usa `.find()` o `.count()` para solo preguntar |
| Esperar orden en un `std::unordered_map` | No lo hay: usa `std::map` u ordena las claves |
| `mapa.at("x")` con una clave que no existe | Lanza `std::out_of_range` |
| Un `double` como clave | Compila, pero las comparaciones de punto flotante son delicadas: evita claves decimales |

## Para practicar

Haz los [ejercicios 3.2 de mapas](mapas.md).
