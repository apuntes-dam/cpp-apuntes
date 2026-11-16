# 7.B Archivos y carpetas

## Rutas con `std::filesystem`

Desde C++17, `<filesystem>` da un tipo `std::filesystem::path` que representa una ruta sin importar el sistema operativo:

```cpp
#include <filesystem>
#include <iostream>
namespace fs = std::filesystem;

int main() {
    fs::path ruta = "datos/clientes/ana.txt";

    std::cout << ruta.filename() << std::endl;       // "ana.txt"
    std::cout << ruta.extension() << std::endl;       // ".txt"
    std::cout << ruta.parent_path() << std::endl;     // "datos/clientes"
    std::cout << std::boolalpha << ruta.is_absolute() << std::endl;

    fs::path nueva = "datos";          // construir una ruta pieza a pieza
    nueva /= "clientes";
    nueva /= "luis.txt";
    std::cout << nueva.string() << std::endl;
    return 0;
}
```

Salida:

```text
"ana.txt"
".txt"
"datos/clientes"
false
datos\clientes\luis.txt
```

`operator/=` (y `operator/`) añaden un trozo de ruta con el separador correcto de cada sistema: nunca concatenes rutas con `+` y una barra escrita a mano.

## Comprobar, crear y borrar

```cpp
#include <filesystem>
#include <fstream>
#include <iostream>
namespace fs = std::filesystem;

int main() {
    fs::create_directories("demo/sub");      // crea «demo» y «demo/sub» de una vez, sin fallar si ya existen
    { std::ofstream f("demo/a.txt"); f << "hola"; }

    std::cout << std::boolalpha << fs::exists("demo/a.txt") << std::endl;
    std::cout << fs::file_size("demo/a.txt") << " bytes, es archivo: " << fs::is_regular_file("demo/a.txt") << std::endl;

    for (const auto& entrada : fs::directory_iterator("demo")) {
        std::cout << entrada.path().string() << std::endl;
    }

    fs::remove("demo/a.txt");
    fs::remove_all("demo");      // borra la carpeta con todo su contenido
    return 0;
}
```

Salida:

```text
true
4 bytes, es archivo: true
demo\a.txt
demo\sub
```

## Controlar los errores con sentido

Las funciones de `<filesystem>` tienen dos versiones: una que **lanza** `std::filesystem::filesystem_error` si algo va mal, y otra que recibe un `std::error_code&` y nunca lanza (tú compruebas el código). Abrir un `std::ifstream`/`std::ofstream` nunca lanza: el objeto queda en estado de error y se comprueba con `if (!flujo)`:

```cpp
#include <filesystem>
#include <fstream>
#include <iostream>
namespace fs = std::filesystem;

int main() {
    std::error_code ec;
    bool existe = fs::exists("no_existe.txt", ec);
    if (!existe) {
        std::cout << "No existe ese archivo" << std::endl;
    }

    std::ifstream f("/ruta/sin/permiso/cualquiera");
    if (!f) {
        std::cout << "No se pudo abrir" << std::endl;
    }
    return 0;
}
```

Salida:

```text
No existe ese archivo
No se pudo abrir
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Construir rutas concatenando texto (`carpeta + "/" + archivo`) | Usa `path::operator/=` o `path::operator/`: ponen el separador correcto en cada sistema |
| `create_directory` cuando la carpeta ya existe | Lanza si no puede crearla; usa `create_directories`, que no falla si ya está creada |
| Olvidar que `remove_all` borra **todo** el contenido | Compruébalo dos veces antes de llamarlo con una ruta variable |
| No comprobar si el `std::ifstream`/`std::ofstream` se abrió | Nunca lanzan al fallar: compruébalo con `if (!flujo)` antes de usarlo |

## Para practicar

Haz los [ejercicios 7.2 de archivos y carpetas](archivos.md).
