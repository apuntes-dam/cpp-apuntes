# 6.D Comentarios y documentación

## Cuándo comentar (y cuándo no)

Un comentario que repite lo que el código ya dice no ayuda: `edad++; // suma uno a la edad` no aporta nada que `edad++` no diga ya. Un comentario vale la pena cuando explica el **por qué** — una decisión no obvia, una restricción, un rodeo necesario por un detalle del lenguaje — algo que el código, por sí solo, no puede contar.

```cpp
// Se compara con una tolerancia porque los double no representan 0.1 de forma exacta
if (std::abs(a - b) < 1e-9) { ... }
```

## Comentarios de documentación (estilo Doxygen)

Para documentar una clase o una función de cara a quien la use (no a quien lee el cuerpo), C++ no tiene una sintaxis propia como `///` en Rust o `/** */` en Java, pero la convención más extendida es la misma sintaxis `/** ... */` que entiende **Doxygen**, la herramienta que genera documentación en HTML a partir de esos comentarios:

```cpp
/**
 * Calcula el precio final tras aplicar un descuento.
 * @param precio Precio original, en euros.
 * @param descuento Objeto que decide cómo se aplica el descuento.
 * @return El precio ya rebajado.
 */
double precioFinal(double precio, const Descuento& descuento);
```

`@param` documenta cada parámetro y `@return` el valor devuelto; Doxygen (`doxygen Doxyfile`, con un fichero de configuración) recorre el código, lee estos bloques y genera páginas HTML navegables con la documentación de cada clase y función, de forma parecida a como `cargo doc` lo hace en Rust o Javadoc en Java. No hace falta instalar nada para **escribir** estos comentarios — solo para generar la documentación final con la herramienta.

## Para practicar

No hay ejercicio específico de esta página.
