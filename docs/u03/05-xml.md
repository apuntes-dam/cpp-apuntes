# 3.5 XML

**XML** es otro formato de texto para guardar datos, con **etiquetas**. La biblioteca estándar de C++ tampoco trae XML; aquí se usa **[tinyxml2](https://github.com/leethomason/tinyxml2)**, ligera y capaz de **leer y escribir** (a diferencia del `roxmltree` de Rust, que solo lee), instalada con vcpkg.

## Partes de un XML

| Parte | Ejemplo |
|---|---|
| **Elemento** | `<nombre>Juan</nombre>` |
| **Elemento raíz** | `<usuarios> ... </usuarios>` (uno solo, que contiene todo) |
| **Atributo** | `<usuario id="1">` |
| **Texto** | `Juan` (lo de dentro de las etiquetas) |

## Leer

Instalado con vcpkg: `tinyxml2`.

```cpp
#include <iostream>
#include <tinyxml2.h>
using namespace tinyxml2;

int main() {
    const char* texto =
        "<usuarios>"
        "<usuario><id>1</id><nombre>Juan</nombre><edad>30</edad></usuario>"
        "<usuario><id>2</id><nombre>Ana</nombre><edad>25</edad></usuario>"
        "</usuarios>";

    XMLDocument doc;
    doc.Parse(texto);
    XMLElement* raiz = doc.RootElement();
    std::cout << raiz->Name() << std::endl;

    for (XMLElement* usuario = raiz->FirstChildElement("usuario"); usuario; usuario = usuario->NextSiblingElement("usuario")) {
        const char* id = usuario->FirstChildElement("id")->GetText();
        const char* nombre = usuario->FirstChildElement("nombre")->GetText();
        int edad = usuario->FirstChildElement("edad")->IntText();
        std::cout << id << " " << nombre << " (" << edad << " años)" << std::endl;
    }
    return 0;
}
```

Salida:

```text
usuarios
1 Juan (30 años)
2 Ana (25 años)
```

`FirstChildElement("nombre")` devuelve `nullptr` si no existe ese hijo — comprueba antes de llamar a `->GetText()` sobre él si no estás seguro de que está.

## Atributos

Instalado con vcpkg: `tinyxml2`.

```cpp
#include <iostream>
#include <tinyxml2.h>
using namespace tinyxml2;

int main() {
    XMLDocument doc;
    doc.Parse(R"(<tienda><producto codigo="A1" precio="2.5">Pan</producto>)"
               R"(<producto codigo="B2" precio="1.2">Leche</producto></tienda>)");

    double total = 0.0;
    for (XMLElement* p = doc.RootElement()->FirstChildElement("producto"); p; p = p->NextSiblingElement("producto")) {
        double precio = p->DoubleAttribute("precio");
        std::cout << p->GetText() << " (" << p->Attribute("codigo") << ") " << precio << std::endl;
        total += precio;
    }
    std::cout << "Total: " << total << std::endl;
    return 0;
}
```

Salida:

```text
Pan (A1) 2.5
Leche (B2) 1.2
Total: 3.7
```

`DoubleAttribute`/`IntAttribute`/`BoolAttribute` leen y convierten a la vez; `Attribute("nombre")` da el texto tal cual (o `nullptr` si no existe).

## Construir y escribir

Instalado con vcpkg: `tinyxml2`.

```cpp
#include <iostream>
#include <tinyxml2.h>
using namespace tinyxml2;

int main() {
    XMLDocument doc;
    XMLElement* raiz = doc.NewElement("usuarios");
    doc.InsertFirstChild(raiz);

    for (auto [id, nombre, edad] : std::initializer_list<std::tuple<int, const char*, int>>{{1, "Juan", 30}, {2, "Ana & Luis", 25}}) {
        XMLElement* usuario = doc.NewElement("usuario");
        XMLElement* elId = doc.NewElement("id");
        elId->SetText(id);
        XMLElement* elNombre = doc.NewElement("nombre");
        elNombre->SetText(nombre);          // tinyxml2 escapa «&», «<»... al escribir: no hay que hacerlo a mano
        XMLElement* elEdad = doc.NewElement("edad");
        elEdad->SetText(edad);
        usuario->InsertEndChild(elId);
        usuario->InsertEndChild(elNombre);
        usuario->InsertEndChild(elEdad);
        raiz->InsertEndChild(usuario);
    }

    XMLPrinter impresora;
    doc.Print(&impresora);
    std::cout << impresora.CStr() << std::endl;
    doc.SaveFile("datos_usuarios_nuevo.xml");
    return 0;
}
```

Salida:

```text
<usuarios>
    <usuario>
        <id>1</id>
        <nombre>Juan</nombre>
        <edad>30</edad>
    </usuario>
    <usuario>
        <id>2</id>
        <nombre>Ana &amp; Luis</nombre>
        <edad>25</edad>
    </usuario>
</usuarios>
```

## Errores: códigos, no excepciones

A diferencia de `nlohmann::json`, **tinyxml2 no lanza excepciones**: cada operación que puede fallar devuelve un `XMLError` (`doc.Parse(...)`, `doc.LoadFile(...)`) que hay que comprobar tú mismo:

Instalado con vcpkg: `tinyxml2`.

```cpp
#include <iostream>
#include <tinyxml2.h>
using namespace tinyxml2;

int main() {
    for (const char* texto : {"<a><b>hola</b></a>", "<a><b>hola</a>", ""}) {
        XMLDocument doc;
        XMLError resultado = doc.Parse(texto);
        if (resultado == XML_SUCCESS) {
            std::cout << "\"" << texto << "\" -> bien formado" << std::endl;
        } else {
            std::cout << "\"" << texto << "\" -> error: " << doc.ErrorStr() << std::endl;
        }
    }
    return 0;
}
```

Salida:

```text
"<a><b>hola</b></a>" -> bien formado
"<a><b>hola</a>" -> error: Error=XML_ERROR_MISMATCHED_ELEMENT ErrorID=14 (0xe) Line number=1: XMLElement name=b
"" -> error: Error=XML_ERROR_EMPTY_DOCUMENT ErrorID=13 (0xd) Line number=0
```

## ¿JSON o XML?

| | JSON | XML |
|---|---|---|
| Soporte en C++ | `nlohmann/json`, de solo cabecera | `tinyxml2`, lee y escribe |
| Errores | Excepciones (`json::parse_error`, `json::type_error`) | Códigos de error (`XMLError`), sin excepciones |
| Tamaño | Más compacto | Más verboso |
| Atributos y texto mezclado | No | Sí |
| Validación con esquema | JSON Schema (aparte) | DTD, XSD (ver la [web de XML](https://apuntes-dam.github.io/xml-apuntes/)) |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a `->GetText()` sobre un `FirstChildElement` que no existe | Devuelve `nullptr`: compruébalo antes, o el programa termina sin aviso claro |
| Esperar que un error de `tinyxml2` lance una excepción | Devuelve un `XMLError`: compruébalo tras cada `Parse`/`LoadFile` |
| Escapar `&`, `<`, `>` a mano al escribir con `SetText` | `tinyxml2` ya lo hace por ti |
| Confundir `IntText()`/`DoubleText()` (contenido) con `IntAttribute()`/`DoubleAttribute()` (un atributo) | El primer grupo lee el **texto** del elemento; el segundo, un **atributo** |

## Para practicar

Haz el [ejercicio 3.5 de XML](xml.md).
