# U7.3 · Ficheros de texto

<div class="ej-gate" data-unit="u07" data-nombre="U7 · Entrada/salida y GUI"></div>

Leer y escribir texto: secuencial, línea a línea, y siempre cerrando el archivo y controlando los errores.

## Ejercicio 7.7

**Contador.** Dado un archivo de texto, muestra su número de líneas, de palabras y de caracteres.

!!! note "En C++"
    Usa `rdbuf()` sobre un `std::ifstream` y `std::getline` sobre el contenido, o lee el archivo directamente línea a línea con `std::getline` si puede ser grande.

<details class="sol" data-key="u69/u7-3/7.7">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>#include &lt;filesystem&gt;
#include &lt;fstream&gt;
#include &lt;iostream&gt;
#include &lt;sstream&gt;
namespace fs = std::filesystem;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.8

**Copia en mayúsculas.** Lee un archivo línea a línea y crea otro con el mismo contenido en mayúsculas. Si el archivo de destino ya existe, pregunta antes de sobrescribirlo.

<details class="sol" data-key="u69/u7-3/7.8">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>#include &lt;algorithm&gt;
#include &lt;filesystem&gt;
#include &lt;fstream&gt;
#include &lt;iostream&gt;
namespace fs = std::filesystem;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.9

**Agenda de contactos persistente.** Guarda contactos en un archivo de texto, una línea por contacto con el formato `nombre;teléfono`. Menú: añadir, buscar por nombre, listar y salir. Al arrancar, carga el archivo (si no existe, empieza vacío); al añadir, guarda. Ignora las líneas mal formadas mostrando un aviso.

## Ejercicio 7.10

**Registro (log).** Escribe una función `registrar(mensaje)` que **añada** al final de un archivo una línea con la fecha y hora (`AAAA-MM-DD HH:MM:SS`) y el mensaje, sin borrar lo anterior. Llámala desde un programa pequeño varias veces y comprueba el archivo.

!!! note "En C++"
    La fecha con ese formato necesita `<chrono>` (`std::chrono::system_clock::now()` con `std::format`, si tu compilador lo soporta) o `<ctime>` con `std::strftime`; para añadir al final, abre el `std::ofstream` con `std::ios::app`.

## Ejercicio 7.11

**Notas desde CSV.** Dado un archivo `notas.csv` con líneas `alumno;nota1;nota2;nota3`, calcula la nota media de cada alumno y la media de la clase, y escribe el resultado en `medias.txt`. Controla las notas que no sean números.
