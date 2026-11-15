# 2.3 Excepciones

A diferencia de Rust, **C++ tiene excepciones de verdad**, parecidas a las de Java, C# o Python: un `throw` interrumpe la función en marcha (y todas las que la llamaron) hasta encontrar un `catch` que las espere. Si nadie las captura, el programa **termina**.

## `try`, `catch` y `throw`

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

int main() {
    for (std::string texto : {"42", "abc", "", "3.5", " 7 "}) {
        try {
            int n = std::stoi(texto);
            std::cout << "\"" << texto << "\" -> " << n << std::endl;
        } catch (const std::invalid_argument& e) {
            std::cout << "\"" << texto << "\" -> error: " << e.what() << std::endl;
        }
    }
    return 0;
}
```

Salida:

```text
"42" -> 42
"abc" -> error: invalid stoi argument
"" -> error: invalid stoi argument
"3.5" -> 3
" 7 " -> 7
```

`std::stoi`/`std::stod` lanzan `std::invalid_argument` si el texto no empieza por un número, y `std::out_of_range` si el número no cabe en el tipo (ambas, declaradas en `<stdexcept>`). `" 7 "` **sí** funciona: `stoi` ignora los espacios iniciales y lo que sobra después del número, sin avisar — por eso conviene validar la entrada por separado si eso te importa.

## Capturar por referencia `const`, y varios tipos

Captura siempre **por referencia `const`** (`catch (const Tipo& e)`), nunca por valor: evita una copia innecesaria y, con jerarquías de excepciones, conserva el tipo real del objeto. Los `catch` se prueban **en orden**, de más concreto a más genérico:

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>

int main() {
    std::vector<int> numeros = {10, 20, 30};
    for (int indice : {1, 10, -1}) {
        try {
            std::cout << numeros.at(indice) << std::endl;   // at() comprueba el índice; [] no
        } catch (const std::out_of_range& e) {
            std::cout << "Índice fuera de rango: " << e.what() << std::endl;
        } catch (const std::exception& e) {                // cualquier otra excepción «de verdad»
            std::cout << "Otro error: " << e.what() << std::endl;
        }
    }
    return 0;
}
```

Salida:

```text
20
Índice fuera de rango: invalid vector subscript
Índice fuera de rango: invalid vector subscript
```

`numeros[indice]` (con corchetes) **no comprueba nada**: un índice fuera de rango es comportamiento indefinido, no una excepción. Usa `.at(indice)` cuando el índice pueda estar mal.

## Lanzar una excepción propia

Para dar a un fallo un nombre y un mensaje propios, se define una clase que hereda de `std::exception` y sobrescribe `what()`:

```cpp
#include <iostream>
#include <string>
#include <exception>

class ContrasenaIncorrecta : public std::exception {
    std::string mensaje;
public:
    explicit ContrasenaIncorrecta(std::string m) : mensaje(std::move(m)) {}
    const char* what() const noexcept override { return mensaje.c_str(); }
};

class UsuarioBloqueado : public std::exception {
    std::string mensaje;
public:
    explicit UsuarioBloqueado(const std::string& quien) : mensaje("El usuario " + quien + " está bloqueado") {}
    const char* what() const noexcept override { return mensaje.c_str(); }
};

void entrar(const std::string& usuario, const std::string& clave) {
    if (usuario == "root") throw UsuarioBloqueado(usuario);
    if (clave != "secreto") throw ContrasenaIncorrecta("Incorrect Password!!");
}

int main() {
    for (auto [u, c] : std::initializer_list<std::pair<std::string, std::string>>{{"ana", "secreto"}, {"ana", "1234"}, {"root", "secreto"}}) {
        try {
            entrar(u, c);
            std::cout << u << ": acceso concedido" << std::endl;
        } catch (const std::exception& e) {
            std::cout << u << ": Error: " << e.what() << std::endl;
        }
    }
    return 0;
}
```

Salida:

```text
ana: acceso concedido
ana: Error: Incorrect Password!!
root: Error: El usuario root está bloqueado
```

`what()` **debe** marcarse `noexcept` (no puede lanzar otra excepción al preguntarle el mensaje) y devuelve un `const char*` válido mientras el objeto exista — por eso `mensaje` es un `std::string` miembro, no un puntero a un texto temporal.

## `catch (...)` y relanzar con `throw;`

`catch (...)` captura **cualquier cosa**, sin poder leer el tipo ni el mensaje — solo úsalo como última red de seguridad. Dentro de un `catch`, un `throw;` sin nada detrás **relanza** la misma excepción, para que la trate quien llamó a esta función:

```cpp
#include <iostream>
#include <stdexcept>

void validar(int n) {
    try {
        if (n < 0) throw std::invalid_argument("no puede ser negativo");
    } catch (...) {
        std::cout << "validar: algo falló, lo paso hacia arriba" << std::endl;
        throw;
    }
}

int main() {
    try {
        validar(-5);
    } catch (const std::exception& e) {
        std::cout << "main: " << e.what() << std::endl;
    }
    return 0;
}
```

Salida:

```text
validar: algo falló, lo paso hacia arriba
main: no puede ser negativo
```

## Si nadie la captura: el programa termina

Una excepción que llega hasta `main` sin un `catch` que la espere **termina el programa** (llamando a `std::terminate`), con un **código de salida distinto de cero** (en algunos entornos, además, un mensaje de depuración):

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

int main() {
    try {
        int n = std::stoi("abc");
        std::cout << "Has escrito " << n << std::endl;
    } catch (const std::invalid_argument& e) {
        std::cout << "La entrada no es correcta" << std::endl;
        throw;   // relanza el error: el programa termina con él
    }
    return 0;
}
```

**Al ejecutar:**

```text
La entrada no es correcta
```

## `finally` no existe: usa destructores (RAII)

C++ no tiene una palabra clave `finally`. En su lugar, cualquier objeto local se **destruye automáticamente** al salir de su bloque, **tanto si sale por el final como por una excepción** — por eso un fichero, un candado o una conexión que se cierran en su destructor se cierran siempre, sin código extra (es el patrón **RAII**, que verás más despacio con las clases, en una unidad futura).

## Cuándo usar excepciones

* Para **errores que el que llama puede y debe tratar**: datos del usuario, un archivo que no existe, una conexión que falla.
* **No** las uses para el flujo normal del programa (por ejemplo, para salir de un bucle): son más lentas que un `if` y dificultan seguir la lógica.
* Un destructor **nunca** debería dejar escapar una excepción: si algo puede fallar al destruir, hay que capturarlo ahí mismo.

## Errores frecuentes

* **Capturar por valor** (`catch (Tipo e)`) en vez de por referencia `const`: copia el objeto y, con una jerarquía de excepciones, puede perder el tipo real.
* **`catch (...)` como primera opción.** Oculta el tipo real del error y hace casi imposible depurarlo; déjalo solo como último recurso.
* **Lanzar un tipo que no hereda de `std::exception`** (por ejemplo, `throw 42;` o `throw "mensaje";`): compila, pero un `catch (const std::exception&)` no lo atrapa.
* **Olvidar `const` y `&` en `catch`.**
* **Dejar escapar una excepción de un destructor.**

## Para practicar

Haz los [ejercicios 2.3 de excepciones](excepciones.md). El objetivo es que **ningún error llegue al programa principal sin tratar** y lo aborte.
