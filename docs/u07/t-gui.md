# 7.D Interfaces gráficas

Hasta ahora los programas **leían** de la consola y **escribían** en ella, en un orden fijo. Una **interfaz gráfica** (GUI) cambia la idea: el programa muestra una **ventana** y **espera** a que el usuario haga algo (escribir, pulsar, elegir). Se dice que está **dirigido por eventos**.

## Ideas comunes a todas las librerías

| Idea | Qué es |
|---|---|
| **Componente** (*widget*) | Cada elemento de la ventana: texto, campo de entrada, botón |
| **Contenedor y diseño** (*layout*) | Cómo se colocan los componentes |
| **Evento** | Algo que ocurre: una pulsación, una tecla, un clic |
| **Bucle de eventos** | El ciclo que espera eventos y los reparte; se pone en marcha al abrir la ventana (`Fl::run()`) |
| **Estado** | Los datos que determinan lo que se ve, y que el programa va actualizando |

El programa no controla el orden: **reacciona**.

## C++ no trae una GUI en la biblioteca estándar

A diferencia de la consola o los archivos, C++ no tiene ninguna interfaz gráfica «de serie»: hace falta elegir una librería de terceros.

| Librería | Qué es | Notas |
|---|---|---|
| **FLTK** | Ligera, pocas dependencias, API sencilla de widgets | La más sencilla para empezar; la usada aquí |
| **Dear ImGui** | Inmediata (se redibuja todo cada fotograma), muy usada en herramientas y videojuegos | Necesita montar tú mismo la ventana (DirectX, OpenGL...) |
| **Qt** | Completa: widgets, estilos, internacionalización, herramientas visuales | Potente, pero pesada de instalar y con su propia licencia |
| **wxWidgets** | Usa los widgets **nativos** del sistema operativo | El aspecto cambia según dónde se ejecute, como el propio sistema |

## Separar la lógica de la ventana

Como en los demás lenguajes del módulo, conviene que la función que **calcula** no sepa nada de ventanas, para poder probarla sin abrir ninguna:

```cpp
#include <sstream>
#include <stdexcept>
#include <string>

std::string calcular_propina(const std::string& texto) {
    try {
        size_t pos;
        double importe = std::stod(texto, &pos);
        if (pos != texto.size()) throw std::invalid_argument("sobra texto");
        if (importe < 0.0) return "El importe no puede ser negativo";
        std::ostringstream out;
        out << std::fixed;
        out.precision(2);
        out << "Propina (10%): " << (importe * 0.10);
        return out.str();
    } catch (...) {
        return "Eso no es un número";
    }
}

#include <iostream>

int main() {
    std::cout << calcular_propina("100") << std::endl;
    std::cout << calcular_propina("-5") << std::endl;
    std::cout << calcular_propina("abc") << std::endl;
    return 0;
}
```

Salida:

```text
Propina (10%): 10.00
El importe no puede ser negativo
Eso no es un número
```

## La ventana

Con `calcular_propina` ya probada, la ventana solo **recoge** el texto, **llama** a la función y **muestra** el resultado:

```cpp
#include <FL/Fl.H>
#include <FL/Fl_Window.H>
#include <FL/Fl_Input.H>
#include <FL/Fl_Button.H>
#include <FL/Fl_Box.H>

Fl_Input* entrada;
Fl_Box* resultado;

void al_pulsar(Fl_Widget*, void*) {
    resultado->copy_label(calcular_propina(entrada->value()).c_str());
}

int main() {
    Fl_Window ventana(300, 150, "Propina");
    entrada = new Fl_Input(90, 20, 180, 30, "Importe:");
    Fl_Button boton(90, 60, 100, 30, "Calcular");
    boton.callback(al_pulsar);
    resultado = new Fl_Box(90, 100, 180, 30);
    ventana.end();
    ventana.show();
    return Fl::run();
}
```

Se compila enlazando con FLTK y se lanza ejecutando el programa. Este ejemplo se ha **comprobado compilándolo de verdad** (con `cl.exe`), pero no se ejecuta aquí: abrir una ventana no tiene sentido al generar una página web, y es justo lo que harías tú al probarlo en tu ordenador.

## Lo que cambia frente a la consola

| Consola | GUI (FLTK) |
|---|---|
| El programa pide datos **en orden** | El usuario decide **cuándo** y **qué** rellenar |
| `std::cin` bloquea hasta que hay respuesta | El `callback` se llama **cuando ocurre el evento**, no espera en medio del programa |
| El estado son variables locales de `main` | El estado vive en los propios *widgets* (aquí, `entrada` y `resultado`) o en variables que sobreviven entre eventos |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Meter la lógica de cálculo dentro del `callback` | Sepárala en una función aparte: se puede probar sin abrir una ventana |
| Guardar un resultado en una variable local de una función que solo se llama una vez | Si otro evento necesita leerlo, tiene que vivir en un *widget* o en una variable que no desaparezca |
| Esperar que el programa se bloquee esperando una pulsación | El control lo lleva `Fl::run()`: tu código solo reacciona en los *callbacks* |

## Para practicar

Haz el [ejercicio 7.4](gui.md) (en tu ordenador, con ventana de verdad).
