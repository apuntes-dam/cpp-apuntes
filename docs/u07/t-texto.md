# 7.C Ficheros de texto

## Leer

```cpp
#include <fstream>
#include <iostream>
#include <sstream>

int main() {
    { std::ofstream f("notas.txt"); f << "primera linea\nsegunda linea\n\nultima\n"; }   // el bloque cierra f al salir

    std::ifstream f("notas.txt");
    std::stringstream buffer;
    buffer << f.rdbuf();                     // todo de una vez
    std::string contenido = buffer.str();
    std::cout << contenido.size() << " caracteres" << std::endl;
    f.close();

    std::ifstream f2("notas.txt");
    std::string linea;
    int i = 0;
    while (std::getline(f2, linea)) {         // línea a línea, sin el salto final
        std::cout << i << ": \"" << linea << "\"" << std::endl;
        i++;
    }

    std::remove("notas.txt");
    return 0;
}
```

Salida:

```text
36 caracteres
0: "primera linea"
1: "segunda linea"
2: ""
3: "ultima"
```

Para archivos **grandes**, en vez de cargarlos enteros en memoria con `rdbuf()`, se leen línea a línea con `std::getline`, igual que en el segundo bucle de arriba: cada vuelta reutiliza la misma variable, sin acumular nada que no haga falta.

## Cerrar el archivo

Un `std::ifstream`/`std::ofstream` es un objeto normal: cuando **sale de ámbito**, su destructor cierra el archivo automáticamente (RAII), igual que ocurre con cualquier otro recurso en C++. Por eso en el ejemplo de arriba el primer bloque `{ std::ofstream f(...); ... }` no necesita un `close()` explícito: termina el bloque y `f` se destruye, cerrando el archivo antes de que el programa siga.

## Escribir y añadir

```cpp
#include <fstream>
#include <iostream>
#include <sstream>

int main() {
    { std::ofstream f("registro.txt"); f << "primera entrada\n"; }       // crea o SOBRESCRIBE

    { std::ofstream f("registro.txt", std::ios::app); f << "segunda entrada\n"; }   // añade al final, sin borrar lo anterior

    std::ifstream f("registro.txt");
    std::stringstream buffer;
    buffer << f.rdbuf();
    std::cout << buffer.str();

    std::remove("registro.txt");
    return 0;
}
```

Salida:

```text
primera entrada
segunda entrada
```

Abrir un `std::ofstream` sin más **sobrescribe** el archivo entero; para añadir hace falta el modo `std::ios::app`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Abrir un `std::ofstream` pensando que añade | Sobrescribe siempre: para añadir, el modo `std::ios::app` |
| Cargar un archivo enorme entero con `rdbuf()` | Para archivos grandes, lee línea a línea con `std::getline` |
| Olvidar que `std::getline` quita el salto de línea de cada línea | Si lo necesitas, añádelo tú al volver a escribir |
| No comprobar si el archivo se abrió | `if (!flujo)` antes de leer o escribir: abrir no lanza, solo deja el flujo en estado de error |

## Para practicar

Haz los [ejercicios 7.3 de ficheros de texto](texto.md).
