# 3.0 Cadenas

En C++ hay, en la práctica, **dos formas de texto**:

| Tipo | Qué es | Se puede cambiar | Ejemplo |
|---|---|---|---|
| **`const char*`** (al estilo C) | Un puntero a los bytes, terminados en `\0` | No, directamente | `"hola"` |
| **`std::string`** | Un objeto que **posee** sus datos y puede crecer | Sí | `std::string s = "hola";` |

Usa **`std::string`** casi siempre: es la que trae operadores, métodos y gestiona su propia memoria. Un literal `"hola"` es compatible con los dos (se convierte a `std::string` automáticamente donde haga falta).

## Recorrer y buscar

```cpp
#include <iostream>
#include <string>
#include <algorithm>

int main() {
    std::string texto = "murciélago";

    // tamaño: size() cuenta BYTES, no «letras» (las tildes en UTF-8 ocupan 2 bytes)
    std::cout << texto.size() << " bytes" << std::endl;

    // recorrer los bytes
    for (size_t i = 0; i < 3; ++i) {
        std::cout << texto[i];
    }
    std::cout << std::endl;

    // dar la vuelta byte a byte solo es seguro con ASCII puro: con una tilde (2 bytes) saldría mal
    std::string ascii = "Hola";
    std::string al_reves(ascii.rbegin(), ascii.rend());
    std::cout << al_reves << std::endl;

    // buscar: find devuelve una posición, o std::string::npos si no está
    std::cout << texto.find('c') << " ";
    std::cout << (texto.find('z') == std::string::npos ? -1 : (int)texto.find('z')) << std::endl;
    std::cout << (texto.find("rci") != std::string::npos) << std::endl;

    // empieza/termina por
    std::cout << (texto.rfind("mur", 0) == 0) << " ";                                   // empieza por «mur»
    std::cout << (texto.size() >= 2 && texto.compare(texto.size() - 2, 2, "go") == 0);   // termina en «go»
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
11 bytes
mur
aloH
3 -1
1
1 1
```

`al_reves` da la vuelta a los **bytes**. Con `ascii` (ASCII puro) coincide con dar la vuelta a las letras, pero sobre `texto` (con la `é` de 2 bytes) **rompería** la cadena: cortaría ese carácter por la mitad y el resultado no sería UTF-8 válido. Para texto con acentos, da la vuelta por **puntos de código** con una biblioteca Unicode.

## Trozos de una cadena

`substr(inicio, longitud)` corta por **posición de byte**. No hay una comprobación automática de límites como en otros lenguajes: pasarte lanza `std::out_of_range` (ver la [unidad 2](../u02/02-excepciones.md)).

```cpp
#include <iostream>
#include <string>

int main() {
    std::string texto = "Hola, mundo";
    std::cout << texto.substr(0, 4) << std::endl;      // "Hola"
    std::cout << texto.substr(6) << std::endl;         // sin longitud: hasta el final
    std::cout << texto.substr(6, 5) << std::endl;       // "mundo"
    return 0;
}
```

Salida:

```text
Hola
mundo
mundo
```

## Transformar

Casi todos los métodos de `<algorithm>` modifican **en el sitio** (no devuelven una copia); los de `std::string` como `substr` sí devuelven una cadena nueva:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <sstream>

int main() {
    std::string s = "  Hola, Mundo  ";

    // recortar espacios (trim): no hay un método trim, se hace a mano
    size_t inicio = s.find_first_not_of(' ');
    size_t fin = s.find_last_not_of(' ');
    std::string recortado = s.substr(inicio, fin - inicio + 1);
    std::cout << "[" << recortado << "]" << std::endl;

    std::string mayus = recortado, minus = recortado;
    std::transform(mayus.begin(), mayus.end(), mayus.begin(), ::toupper);
    std::transform(minus.begin(), minus.end(), minus.begin(), ::tolower);
    std::cout << mayus << " " << minus << std::endl;

    std::string reemplazo = recortado;
    size_t pos = 0;
    while ((pos = reemplazo.find('o', pos)) != std::string::npos) {
        reemplazo.replace(pos, 1, "0");
        pos += 1;
    }
    std::cout << reemplazo << std::endl;

    // dividir por un separador (no hay split en la biblioteca estándar)
    std::string csv = "ana,luis,eva";
    std::vector<std::string> partes;
    std::stringstream flujo(csv);
    std::string parte;
    while (std::getline(flujo, parte, ',')) partes.push_back(parte);
    std::cout << partes.size() << ": " << partes[0] << ", " << partes[1] << ", " << partes[2] << std::endl;
    return 0;
}
```

Salida:

```text
[Hola, Mundo]
HOLA, MUNDO hola, mundo
H0la, Mund0
3: ana, luis, eva
```

## Construir una cadena

```cpp
#include <iostream>
#include <string>
#include <sstream>

int main() {
    std::string s;
    s += "Hola";
    s += ',';
    s += " mundo";
    std::cout << s << std::endl;

    std::string nombre = "Ana";
    int edad = 20;
    std::ostringstream frase;
    frase << nombre << " tiene " << edad << " años";    // concatenar con un flujo evita conversiones a mano
    std::cout << frase.str() << std::endl;

    std::string repetido;
    for (int i = 0; i < 3; ++i) repetido += "ab";
    std::cout << repetido << std::endl;
    return 0;
}
```

Salida:

```text
Hola, mundo
Ana tiene 20 años
ababab
```

## Comparar y convertir

`==` compara el **contenido**. Para ignorar las mayúsculas, pasa las dos a minúsculas antes de comparar:

```cpp
#include <iostream>
#include <string>
#include <algorithm>
#include <cctype>

std::string minusculas(std::string s) {
    std::transform(s.begin(), s.end(), s.begin(), ::tolower);
    return s;
}

int main() {
    std::string a = "C++";
    std::cout << (a == "C++") << std::endl;
    std::cout << (minusculas("Ana") == minusculas("ANA")) << std::endl;
    std::cout << ("a" < "b") << std::endl;   // orden alfabético por bytes

    // texto <-> número
    int n = std::stoi("42");
    std::cout << n + 1 << std::endl;
    std::cout << std::to_string(3.5) + "!" << std::endl;
    std::cout << (std::isalpha('a') && std::isdigit('7')) << std::endl;
    std::cout << (int)'a' << std::endl;
    return 0;
}
```

Salida:

```text
1
1
1
43
3.500000!
1
97
```

## Unicode

`size()` cuenta **bytes**, no letras: en UTF-8, `"año".size()` es 4 (la `ñ` ocupa dos). C++ no trae en su biblioteca estándar una forma sencilla de contar «letras que ves», ni de pasar a mayúsculas texto con tildes (`::toupper` solo entiende ASCII) — para eso, en un proyecto real, se recurre a una biblioteca como ICU. Para el trabajo de estas unidades, con texto simple, basta con tener presente la diferencia entre bytes y caracteres.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `texto.substr(a, b)` con `a` fuera de rango | Lanza `std::out_of_range`: comprueba `a <= texto.size()` antes |
| Suponer que `size()` cuenta letras | Cuenta **bytes**: con acentos UTF-8 son más bytes que letras |
| Esperar que `std::transform(..., ::toupper)` cambie una variable aparte | Modifica **en el sitio** el rango que le pasas: trabaja sobre una copia si quieres conservar la original |
| Olvidar que no hay `split` en la biblioteca estándar | Usa `std::stringstream` + `std::getline(flujo, parte, separador)` |
| Comparar `const char*` con `==` | Compara **punteros**, no contenido: usa `std::string` o `strcmp` |
| `texto[pos]` con `pos` fuera de rango | Comportamiento indefinido (no avisa): usa `.at(pos)` si `pos` puede estar mal |

## Para practicar

Haz los [ejercicios 3.0 de cadenas](cadenas.md).
