# U4 · Programación orientada a objetos

Pasar del código suelto a **clases y objetos**: atributos, métodos, constructores, validación, encapsulamiento y colecciones de objetos. Termina con un proyecto personal o en grupo.

## Teoría

* [4.A Clases y objetos](t-clases.md): atributos, métodos, constructor.
* [4.B Encapsulamiento](t-encapsulamiento.md): visibilidad, validación, `operator==`/`<=>`.
* [4.C Constructores y enumerados](t-constructores.md): sobrecarga, valores por defecto y `enum class`.
* [4.D Colecciones y miembros estáticos](t-colecciones.md): objetos que contienen objetos y lo que pertenece a la clase.
* [4.E Del problema al programa](t-modelar.md): cómo decidir las clases y pasar comportamiento como parámetro.

## Antes de empezar: qué debes dominar

* Clase, objeto, atributo y método.
* Constructores (principal y alternativos) y valores por defecto.
* Encapsulamiento: visibilidad y propiedades de solo lectura.
* Enumerados, sobrecarga y `toString`/igualdad.
* Colecciones de objetos y validación con excepciones.

## Ejercicios de la unidad

| Bloque | Ejercicios |
|---|---|
| [U4.1 · Repaso de las unidades 1 a 3](repaso.md) | 1 |
| [U4.2 · POO I (ejercicios 1 al 5)](poo-1.md) | 5 |
| [U4.3 · POO II (ejercicios 6 al 10)](poo-2.md) | 5 |
| [U4.4 · Robots (parte 1)](robots-1.md) | 2 |
| [U4.5 · Robots (parte 2 y reto)](robots-2.md) | 1 |
| [U4.6 · Prueba: Cafetera y Taza](prueba.md) | 2 |
| [U4.7 · Juego del ahorcado (grupos)](proyecto.md) | 1 |
| [U4.8 · Cambio de rol: explícamelo tú (grupos)](cambio-de-rol.md) | 2 |

## Equivalencias de POO en C++

| Idea | En C++ |
|---|---|
| Constructor | `Persona(std::string nombre, int edad) : nombre(std::move(nombre)), edad(edad) {}` (lista de inicialización); varios con distinta lista de parámetros (sobrecarga) |
| Valores por defecto | `Pizza(Tamano t, std::string nombre = "Margarita")` |
| Propiedad calculada de solo lectura | Un método normal: `double imc() const { return peso / (altura * altura); }` |
| Atributo que solo cambia desde dentro | Campo `private` con un método público que lo lee, y otro, también público o no, que lo cambia validando |
| Privado / público | `private` (y `protected`, `public`); en `class` todo es `private` por defecto, en `struct` todo es `public` |
| Validar al crear | `if (...) throw std::invalid_argument("...");` en el constructor |
| Herencia | `class Hija : public Base`, con la lista de inicialización llamando al constructor de la base; `virtual` en la base y `override` en la hija |
| Clase abstracta / interfaz | Una clase con al menos un método **puro virtual** (`= 0`); si **todos** lo son, es el equivalente a una interfaz |
| Enumerado | `enum class Color { Blanco, Negro };` (con ámbito propio, sin conversión implícita a `int`) |
| Datos con igualdad por valor | `auto operator<=>(const T&) const = default;` (C++20: genera `==`, `<`, `>`... a la vez) |
| Jerarquía cerrada | `final` en la clase (cierra solo esa), o `std::variant`/`std::visit` en vez de herencia (ver 5.D) |
| Representación en texto | Sobrecargar `operator<<(std::ostream&, const T&)` |
| Igualdad | `operator==` a mano, o `<=>` con `= default` |

## Antes de pasar a los ejercicios

Cuando hayas leído y practicado la teoría, marca la casilla para **desbloquear** los ejercicios de esta unidad:

<div class="ej-check" data-unit="u04"></div>
