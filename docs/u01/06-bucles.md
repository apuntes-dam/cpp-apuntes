# 1.6 Bucles: `for`, `while` y `do-while`

Un bucle repite un bloque de código. En C++ hay tres formas: `for`, `while` y `do-while`.

## `for`: la forma clásica, con contador

```cpp
#include <iostream>

int main() {
    for (int i = 1; i <= 5; i++) {
        std::cout << "Numero: " << i << std::endl;
    }
    return 0;
}
```

Salida:

```text
Numero: 1
Numero: 2
Numero: 3
Numero: 4
Numero: 5
```

| Parte | Qué hace |
|---|---|
| `int i = 1` | Se ejecuta **una vez**, al empezar |
| `i <= 5` | Se comprueba **antes** de cada vuelta; si es falsa, el bucle termina |
| `i++` | Se ejecuta **al final** de cada vuelta |
| `{ ... }` | Lo que se repite |

## `for` sobre una colección (`for` basado en rango)

```cpp
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> nombres = {"Ana", "Luis", "Eva"};
    for (const std::string& nombre : nombres) {
        std::cout << nombre << std::endl;
    }
    std::cout << nombres.size() << std::endl;
    return 0;
}
```

Salida:

```text
Ana
Luis
Eva
3
```

`for (const T& elemento : coleccion)` recorre cada elemento **sin copiarlo** (gracias a la referencia `&`) y sin poder modificar la colección (gracias a `const`). Para modificar cada elemento al recorrer, se usa `T&` sin `const`; para trabajar con una copia (tipos pequeños, como `int`), basta `for (int x : numeros)`.

## Variantes del contador

```cpp
#include <iostream>

int main() {
    for (int i = 5; i >= 1; i--) std::cout << i << " ";   // descendente: 5 4 3 2 1
    std::cout << std::endl;
    for (int i = 1; i <= 10; i += 2) std::cout << i << " ";  // de dos en dos: 1 3 5 7 9
    std::cout << std::endl;
    return 0;
}
```

Salida:

```text
5 4 3 2 1 
1 3 5 7 9 
```

Para recorrer con índice **y** elemento a la vez, se combina un contador con el acceso por posición:

```cpp
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> nombres = {"Ana", "Luis", "Eva"};
    for (std::size_t i = 0; i < nombres.size(); i++) {
        std::cout << i << ": " << nombres[i] << std::endl;
    }
    return 0;
}
```

Salida:

```text
0: Ana
1: Luis
2: Eva
```

`std::size_t` es el tipo (sin signo) que usan los tamaños y los índices de la biblioteca estándar; comparar un `int` con signo con un `size_t` sin signo puede dar avisos del compilador, así que para el índice se usa directamente `std::size_t`.

## Comparación rápida

| Característica | `for` | `while` | `do-while` |
|---|---|---|---|
| Cuándo | Se sabe de antemano cuántas vueltas (o se recorre algo) | Repetir mientras se cumpla algo | Repetir **al menos una vez**, y luego mientras se cumpla algo |
| Condición | Antes de cada vuelta | Antes de cada vuelta | **Después** de cada vuelta |
| `break` / `continue` | Sí | Sí | Sí |

## `while` y `do-while`

```cpp
#include <iostream>

int main() {
    int cuenta = 3;
    while (cuenta > 0) {          // comprueba antes: si empezara en 0, no se ejecutaría ninguna vez
        std::cout << cuenta << std::endl;
        cuenta--;
    }

    int intentos = 0;
    do {                          // comprueba después: se ejecuta siempre al menos una vez
        intentos++;
        std::cout << "Intento " << intentos << std::endl;
    } while (intentos < 3);
    return 0;
}
```

Salida:

```text
3
2
1
Intento 1
Intento 2
Intento 3
```

`do { ... } while (condición);` es la única forma que **garantiza** al menos una ejecución, útil por ejemplo para pedir un dato por teclado y repetir mientras no sea válido (unidad 2).

## Cortar un bucle con una variable de control

Una forma sencilla de parar un bucle sin `break`: una variable booleana (una *bandera*) que el bucle consulta en su condición. Cuando pasa lo que buscas, la pones en `false`.

```cpp
#include <iostream>

int main() {
    bool activo = true;
    int i = 1;

    while (activo) {
        std::cout << i << std::endl;
        if (i == 5) {
            activo = false;   // se cumple la condición: el bucle se detiene
        }
        i++;
    }
    return 0;
}
```

Salida:

```text
1
2
3
4
5
```

??? note "Opcional: `break` y `continue`"
    Existen y se usan con normalidad en C++ (más que en otros lenguajes de este sitio), pero el proyecto sigue preferiendo la variable de control cuando el motivo de parar no es evidente a simple vista.

    ```cpp
    #include <iostream>

    int main() {
        for (int i = 1; i <= 10; i++) {
            if (i == 3) continue;   // salta esta vuelta
            if (i == 6) break;      // sale del bucle
            std::cout << i << " ";   // 1 2 4 5
        }
        std::cout << std::endl;
        return 0;
    }
    ```

    Salida:

    ```text
    1 2 4 5 
    ```
