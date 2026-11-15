# 3.4 JSON

**JSON** es un formato de texto para guardar e intercambiar datos. A diferencia de Python o C#, **la biblioteca estándar de C++ no trae JSON**: se usa una biblioteca de terceros. La más extendida es **[nlohmann/json](https://github.com/nlohmann/json)** (de solo cabecera: basta con `#include`, sin compilar nada aparte), instalada aquí con [vcpkg](https://vcpkg.io/).

## Qué contiene un JSON

| Elemento | Ejemplo | En `nlohmann::json` |
|---|---|---|
| Objeto | `{"nombre": "Ana"}` | Como un `std::map<std::string, json>` |
| Lista | `[1, 2, 3]` | Como un `std::vector<json>` |
| Texto | `"hola"` | `std::string` |
| Número | `30`, `1.5` | `int`/`double` |
| Booleano | `true` | `bool` |
| Nulo | `null` | `nullptr` |

## Leer, sin tipos propios

`nlohmann::json` funciona como un **valor dinámico**: los corchetes `[]` acceden por clave o por posición, y convierte sola al tipo que le pidas:

Instalado con vcpkg: `nlohmann-json`.

```cpp
#include <iostream>
#include <nlohmann/json.hpp>
using json = nlohmann::json;

int main() {
    std::string texto = R"({"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]})";
    json datos = json::parse(texto);

    std::cout << datos["usuarios"][0]["nombre"] << std::endl;              // "Juan" (con comillas: es el dump de un json)
    std::cout << datos["usuarios"][1]["edad"] << std::endl;
    std::cout << datos["usuarios"][0]["nombre"].get<std::string>() << std::endl;   // sin comillas
    std::cout << datos["usuarios"].size() << std::endl;
    std::cout << datos.contains("no_existe") << std::endl;                  // false: no lanza nada

    for (const auto& u : datos["usuarios"]) {
        std::cout << u["id"] << ": " << u["nombre"].get<std::string>() << std::endl;
    }
    return 0;
}
```

Salida:

```text
"Juan"
25
Juan
2
0
1: Juan
2: Ana
```

`datos["clave"]` sobre un objeto que **no es `const`** crea la clave con valor `null` si no existe (igual que `std::map`); para preguntar sin crear nada, usa `.contains("clave")`.

## Construir, modificar y escribir

Instalado con vcpkg: `nlohmann-json`.

```cpp
#include <iostream>
#include <nlohmann/json.hpp>
using json = nlohmann::json;

int main() {
    json datos = json::parse(R"({"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]})");

    // modificar
    datos["usuarios"][1]["edad"] = 26;
    datos["usuarios"].push_back({{"id", 3}, {"nombre", "Eva"}, {"edad", 41}});

    // borrar al usuario con id 1: json no tiene «retain», se hace a mano con erase
    auto& lista = datos["usuarios"];
    for (auto it = lista.begin(); it != lista.end(); ) {
        if ((*it)["id"] == 1) it = lista.erase(it); else ++it;
    }

    std::cout << datos.dump() << std::endl;         // compacto
    std::cout << datos.dump(2) << std::endl;        // con sangrado de 2 espacios
    return 0;
}
```

Salida:

```text
{"usuarios":[{"edad":26,"id":2,"nombre":"Ana"},{"edad":41,"id":3,"nombre":"Eva"}]}
{
  "usuarios": [
    {
      "edad": 26,
      "id": 2,
      "nombre": "Ana"
    },
    {
      "edad": 41,
      "id": 3,
      "nombre": "Eva"
    }
  ]
}
```

## Campos que pueden faltar, y archivos

Leer un campo con `[]` que no existe, o con el tipo equivocado, **lanza una excepción** (`json::out_of_range`, `json::type_error`): trátalo con `try`/`catch` (ver la [unidad 2](../u02/02-excepciones.md)) o con `.value("clave", porDefecto)`, que da un valor por defecto sin lanzar nada:

Instalado con vcpkg: `nlohmann-json`.

```cpp
#include <iostream>
#include <fstream>
#include <nlohmann/json.hpp>
using json = nlohmann::json;

json cargar(const std::string& ruta) {
    std::ifstream archivo(ruta);
    if (!archivo) throw std::runtime_error("no se pudo abrir " + ruta);
    json datos;
    archivo >> datos;      // puede lanzar json::parse_error si el contenido no es JSON válido
    return datos;
}

int main() {
    std::ofstream(("datos_usuarios_orig.json")) << R"({"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}]})";

    try {
        json datos = cargar("datos_usuarios_orig.json");
        std::cout << "Guardados " << datos["usuarios"].size() << " usuarios" << std::endl;
        std::cout << datos["usuarios"][0].value("apodo", "(sin apodo)") << std::endl;
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }

    try {
        cargar("no_existe.json");
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }

    std::ofstream(("roto.json")) << R"({ "usuarios": [ )";
    try {
        cargar("roto.json");
    } catch (const std::exception& e) {
        std::cout << "Error: parse" << std::endl;
    }
    return 0;
}
```

Salida:

```text
Guardados 1 usuarios
(sin apodo)
Error: no se pudo abrir no_existe.json
Error: parse
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Un campo con otro tipo del esperado (`"edad": "30"`) | `.get<int>()` lanza `json::type_error`: captura la excepción |
| Leer un campo que puede faltar con `[]` y sin comprobar | Usa `.value("clave", porDefecto)` o `.contains("clave")` antes |
| `datos["clave"]` para **solo preguntar** si existe | Si `datos` no es `const`, la **crea**: usa `.contains()` |
| Olvidar `#include <nlohmann/json.hpp>` | Es de solo cabecera: no hace falta enlazar nada, solo incluirla |
| Un JSON mal formado | `json::parse` lanza `json::parse_error`: captúralo |

## Para practicar

Haz el [ejercicio 3.4 de JSON](json.md).
