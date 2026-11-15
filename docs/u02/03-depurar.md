# 2.4 Depurar: encontrar los errores

**Depurar** es buscar y corregir los errores (*bugs*) de un programa. Los hay de tres clases:

| Tipo | Cuándo aparece | Ejemplo |
|---|---|---|
| **Compilación** | Al compilar | Una llave sin cerrar, un `;` olvidado, un tipo que no encaja |
| **Ejecución** | Con el programa en marcha | Una excepción sin capturar, una división entre cero, un acceso fuera de rango |
| **Lógicos** | Nunca avisan: el resultado es **incorrecto** | Un bucle que se queda corto |

El compilador de MSVC atrapa los errores de compilación, pero sus mensajes a veces son largos y confusos: **lee la primera línea del error primero**, suele ser la más útil, y las siguientes a menudo son consecuencia de la primera.

## Un error que no avisa

Este programa debería sumar los números del 1 al 5 (15), pero **compila, no falla y da otro resultado**:

```cpp
#include <iostream>

int main() {
    int total = 0;
    for (int i = 1; i < 5; ++i) {    // ¡el límite < 5 NO incluye el 5!
        total += i;
    }
    std::cout << total << std::endl;   // esperaba 15
    return 0;
}
```

Salida:

```text
10
```

Se corrige con `i <= 5`. Los errores de «uno de más o de menos» (*off by one*) son los más típicos de los bucles.

## Mensajes por pantalla: `std::cout`, `std::cerr` y `assert`

La técnica más simple: imprimir los valores en los puntos clave. Separa la salida **normal** (`std::cout`) de los **avisos de depuración** (`std::cerr`), para poder quitarlos filtrando una sola secuencia, y recuerda `assert()` de la [unidad 1](../u01/04-pruebas.md) para comprobaciones que deben ser ciertas:

```cpp
#include <iostream>
#include <cassert>

int main() {
    int total = 0;
    for (int i = 1; i <= 5; ++i) {
        int doble = i * 2;
        std::cerr << "[depuración] i=" << i << " doble=" << doble << std::endl;   // no sale en el bloque de arriba: es stderr
        total += doble;
    }
    assert(total == 30);     // comprobación que debe ser cierta; si falla, el programa aborta aquí mismo
    std::cout << total << std::endl;
    return 0;
}
```

Salida:

```text
30
```

## El depurador de Visual Studio

Un **depurador** ejecuta el programa **paso a paso** y deja mirar las variables en cada momento:

1. **Punto de ruptura** (*breakpoint*): haz clic en el margen, junto al número de línea; el programa se detiene ahí.
2. **Ejecuta en modo depuración** (F5, no Ctrl+F5).
3. **Avanza**: *Step Over* (F10, la línea entera), *Step Into* (F11, entra en la función que llama), *Step Out* (sale de ella) y *Continue* (F5, hasta el siguiente punto).
4. **Observa** la ventana *Autos*/*Locals*: cómo cambian las variables en cada paso.
5. Se pueden **evaluar expresiones** en la ventana *Watch* y poner **condiciones** a un punto de ruptura (clic derecho sobre él → *Conditions*).

En VS Code, la extensión *C/C++* de Microsoft (con el mismo depurador, `cppvsdbg` en Windows) ofrece lo mismo.

## Avisos del compilador: `/W4`

Con la bandera `/W4` (nivel de avisos más alto habitual), MSVC señala cosas que compilan pero que probablemente sean un error: una variable sin usar, una comparación con signos distintos, un *fallthrough* en un `switch` sin marcar. Revisa los avisos: a menudo son el bug antes de que lo notes.

## Depurador o mensajes en pantalla

| Mensajes (`std::cout`, `std::cerr`) | Depurador |
|---|---|
| Rápidos de poner, funcionan en cualquier sitio | Control total: pausa, mira y cambia cosas |
| Hay que recompilar y limpiar después | No hay que tocar el código |
| Bien para ver un valor concreto | Mejor para entender **qué camino** sigue el programa |

## Errores típicos en los bucles

| Síntoma | Causa habitual |
|---|---|
| Falta el último elemento, o sobra uno | El límite está mal en un punto (`<` frente a `<=`) |
| El programa **no termina** | La variable de control nunca cambia, o la condición nunca se vuelve falsa |
| El bucle no se ejecuta nunca | La condición es falsa desde el principio |
| El resultado sale solo con la primera vuelta | Falta actualizar un acumulador, o está dentro del bloque equivocado |
| El programa **se cierra sin avisar** (o con una ventana del sistema) | Un acceso fuera de rango con `[]` (no con `.at()`): es comportamiento indefinido, no una excepción |

## Para practicar

El ejercicio [2.4 de depuración](depurar.md) te pide implementar el algoritmo de la burbuja y **depurarlo** paso a paso.
