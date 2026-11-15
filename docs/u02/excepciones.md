# P2.3 · Captura de excepciones

<div class="ej-gate" data-unit="u02" data-nombre="U2 · Estructuras de control"></div>

Salvo que se diga lo contrario, el objetivo es **que ninguna excepción llegue al programa principal y lo aborte**. Controla cualquier dato incorrecto que pueda introducir el usuario.

## Ejercicio 2.3.1

Pide la edad y muestra todos los años cumplidos (de 1 hasta su edad). Controla que lo escrito sea un número.

## Ejercicio 2.3.2

Pide un entero positivo y muestra los impares de 1 hasta ese número, separados por comas. Controla las entradas incorrectas.

## Ejercicio 2.3.3

Pide un entero positivo y muestra la cuenta atrás hasta cero. Repite la petición hasta que el número sea correcto.

## Ejercicio 2.3.4

Pide un entero. Si la entrada no es correcta, muestra `La entrada no es correcta` y vuelve a lanzar la excepción capturada.

!!! note "En C++"
    Usa `std::stoi` dentro de un `try`, y captura `std::invalid_argument`. Para «volver a lanzar» el error, usa `throw;` dentro del `catch` sin capturarlo en una variable (o captúralo con `const std::exception& e` y vuelve a lanzarlo con `throw;`).

<details class="sol" data-key="p2-3/2.3.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>#include &lt;iostream&gt;
#include &lt;string&gt;
std::string leer(const std::string&amp; mensaje) {
    std::cout &lt;&lt; mensaje;
    std::string texto;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 2.3.5

Pide una contraseña y, si no coincide con la guardada, lanza una excepción propia con el mensaje `Incorrect Password!!`.

!!! note "En C++"
    Define una clase que herede de `std::exception` y sobrescriba `what()` (devolviendo un `const char*` válido, no un `std::string` temporal), y que `comprobar` la lance con `throw`. Captúrala con `catch (const TuExcepcion& e)`.

<details class="sol" data-key="p2-3/2.3.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>#include &lt;iostream&gt;
#include &lt;string&gt;
std::string leer(const std::string&amp; mensaje) {
    std::cout &lt;&lt; mensaje;
    std::string texto;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
