# C++

Apuntes personales para aprender **C++**, organizados con la misma estructura que la unidad 1 de Programación: del concepto de programa hasta compilar, ejecutar y comprobar código.

!!! info "Cómo está organizado"
    Cada apartado responde a una pregunta: *qué es un programa*, *cómo se escribe en C++*, *qué datos maneja*, *cómo se comprueba* y *cómo se compila y ejecuta*.

| Apartado | Contenido |
|---|---|
| [U1. Primer programa en C++](u01/index.md) | Estructura, variables, operadores, tipos, pruebas |
| [U2. Estructuras de control](u02/index.md) | Condicionales (`if`, `switch`), excepciones (`try`/`catch`/`throw`) y depurar con Visual Studio |
| [U3. Estructuras de datos](u03/index.md) | Cadenas, `vector`, mapas, conjuntos, JSON (`nlohmann::json`) y XML (`tinyxml2`) |
| [U4. Clases y objetos](u04/index.md) | Clases, encapsulamiento, constructores y `enum class` |
| [U5. Herencia y polimorfismo](u05/index.md) | Herencia, clases abstractas, interfaces y jerarquías cerradas con `final`/`variant` |
| [U6. Diseño con clases y SOLID](u06/index.md) | Diseñar una jerarquía, los cinco principios SOLID y documentar con Doxygen |
| [U7. Entrada/salida y GUI](u07/index.md) | Consola, archivos, ficheros de texto e interfaces gráficas con FLTK |
| [U9. Bases de datos relacionales](u09/index.md) | SQLiteCpp: conexión, consultas, transacciones y el patrón DAO |
| [Entorno](u01/05-entorno.md) | MSVC (`cl.exe`), y las alternativas libres (MinGW-w64/g++, clang) |
| [Prácticas](u01/practicas.md) | Guía de prácticas de la unidad 1 |

!!! note "Qué es C++"
    C++ es un lenguaje **compilado, de tipado estático**, sin recolector de basura: el programador controla directamente la memoria (con ayuda de clases que la gestionan por ti, como `std::string` o `std::vector`). Nació como una extensión de C con clases y hoy añade plantillas genéricas, excepciones y una amplia biblioteca estándar. Se usa en sistemas operativos, motores de videojuegos, aplicaciones de alto rendimiento y mucho software de escritorio.

!!! warning "Web en construcción"
    Por ahora están las **unidades 1 a 7 y la 9** (el material original no tiene unidad 8). Avanzado y el hub llegarán en próximas fases.
