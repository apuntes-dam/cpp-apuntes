# 2.1 Condicionales: decidir qué hacer

Un programa no ejecuta siempre las mismas líneas: **decide** según los datos. Para decidir usa una **condición**, una expresión cuyo resultado es verdadero o falso (`bool`), y las estructuras `if` y `switch`.

## `if`, `else if` y `else`

La estructura completa tiene tres partes: lo que se hace **si** se cumple la condición, lo que se hace si **no** pero se cumple otra, y lo que se hace en **cualquier otro caso**.

```cpp
#include <iostream>
#include <string>

std::string calificacion(int nota) {
    if (nota < 0 || nota > 10) return "Nota no válida";
    else if (nota < 5) return "Insuficiente";
    else if (nota < 6) return "Suficiente";
    else if (nota < 7) return "Bien";
    else if (nota < 9) return "Notable";
    else return "Sobresaliente";
}

int main() {
    for (int nota : {3, 5, 6, 8, 10, 11}) {
        std::cout << nota << ": " << calificacion(nota) << std::endl;
    }
    return 0;
}
```

Salida:

```text
3: Insuficiente
5: Suficiente
6: Bien
8: Notable
10: Sobresaliente
11: Nota no válida
```

Se evalúan **en orden** y solo se ejecuta la **primera** rama que se cumple, por eso `nota < 6` no necesita comprobar también `nota >= 5`. A diferencia de C o de versiones antiguas de otros lenguajes, en C++ moderno `if (nota)` con un entero **sí compila** (cualquier valor distinto de `0` es verdadero), pero es un hábito a evitar: escribe siempre la comparación completa (`if (nota != 0)`).

## Operadores de comparación y lógicos

| Operador | Significa | Ejemplo |
|---|---|---|
| `==` `!=` | igual, distinto | `edad == 18` |
| `<` `>` `<=` `>=` | menor, mayor, menor o igual, mayor o igual | `nota >= 5` |
| `&&` | **y** (las dos condiciones) | `edad >= 18 && edad < 65` |
| `\|\|` | **o** (al menos una) | `dia == "sábado" \|\| dia == "domingo"` |
| `!` | **no** (invierte) | `!terminado` |

`&&` y `||` **cortocircuitan**: si con el primer lado ya se sabe el resultado, el segundo ni se evalúa (`n != 0 && 10 / n > 1` no divide nunca entre cero).

## El operador ternario `?:`

Para una decisión corta entre dos **valores**, el operador `?:` evita un `if`/`else` completo:

```cpp
#include <iostream>

int main() {
    int edad = 20;
    std::string tipo = edad >= 18 ? "adulto" : "menor";
    std::cout << "Eres " << tipo << std::endl;

    int n = -7;
    int valor_absoluto = n < 0 ? -n : n;
    std::cout << valor_absoluto << std::endl;
    return 0;
}
```

Salida:

```text
Eres adulto
7
```

Las dos ramas deberían dar un tipo compatible; si no, el compilador las convierte (o avisa, o da un resultado que no esperabas) — para decisiones con más de un paso, usa `if`/`else`.

## Selección múltiple con `switch`

Cuando comparas **una misma variable con muchos valores concretos** (enteros, `char`, un `enum`), `switch` es más claro que una cadena de `else if`. Cada `case` necesita su propio `break`, o la ejecución **continúa** con el siguiente (*fallthrough*):

```cpp
#include <iostream>

int dias_del_mes(int mes) {
    switch (mes) {
        case 2:
            return 28;
        case 4: case 6: case 9: case 11:   // varios casos seguidos: caen en el mismo return
            return 30;
        default:
            return (mes >= 1 && mes <= 12) ? 31 : 0;
    }
}

int main() {
    std::cout << dias_del_mes(2) << " " << dias_del_mes(4) << " " << dias_del_mes(8) << " " << dias_del_mes(13) << std::endl;

    // el fallthrough intencionado: varios niveles con el mismo mensaje
    for (int nivel : {1, 2, 3, 4}) {
        switch (nivel) {
            case 1:
            case 2:
                std::cout << nivel << ": principiante" << std::endl;
                break;
            case 3:
                std::cout << nivel << ": intermedio" << std::endl;
                break;
            default:
                std::cout << nivel << ": avanzado" << std::endl;
        }
    }
    return 0;
}
```

Salida:

```text
28 30 31 0
1: principiante
2: principiante
3: intermedio
4: avanzado
```

!!! warning "El `switch` NO comprueba que cubras todos los casos"
    A diferencia de Rust (`match`) o Kotlin (`when` como expresión), el compilador de C++ **no exige** un `default`, y **olvidar un `break`** no es un error: simplemente cae al siguiente `case` sin avisar (aunque `/W4` sí avisa de un *fallthrough* sin comentario `[[fallthrough]]`). Pon siempre un `default` y revisa cada `case` al añadir uno nuevo.

`switch` solo compara con **igualdad** y solo acepta tipos enteros, `char` o `enum` (no `std::string`, no rangos como `r if r < 10`): para esos casos, sigue con `if`/`else if`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Condiciones en mal orden, y una rama nunca se ejecuta | Prueba con un valor de cada rama y con los valores **límite** |
| Confundir `=` con `==` | Léelo en voz alta: «es igual a». `if (x = 5)` **compila** (asigna y usa el resultado), así que este error no siempre avisa: con `/W4` el compilador sí lo marca |
| Una condición que no es realmente un `bool` | Escribe la comparación completa: `if (n != 0)` en vez de `if (n)` |
| Olvidar el `break` en un `switch` | Revisa cada `case`; usa `[[fallthrough]];` si es intencionado, para que quede claro |
| Comparar `std::string` con `==` sin tener en cuenta mayúsculas | Pasa las dos a minúscula antes de comparar (ver [3.0](../u03/00-cadenas.md)) |
| Comparar decimales con `==` | `0.1 + 0.2 == 0.3` es `false`: compara con una tolerancia (`std::abs(a - b) < 1e-9`) |

## Para practicar

Haz los [ejercicios 2.1 de condicionales](condicionales.md). Empieza por los primeros y, cuando funcionen, prueba cada uno con un valor de cada rama y con los límites.
