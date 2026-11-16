# 9.A Conectar y crear tablas

## SQLiteCpp

C++ no trae soporte de bases de datos en la biblioteca estándar. Para SQLite se usa **SQLiteCpp**, un envoltorio sobre la librería C de SQLite que cambia los códigos de error por **excepciones** y los punteros por objetos con destructor (RAII), con una API mucho más cómoda que la API C a pelo:

```cpp
// deps: sqlite
#include <SQLiteCpp/SQLiteCpp.h>
```

## Conectar

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <filesystem>
#include <iostream>

int main() {
    {
        SQLite::Database db("tienda.db", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);   // crea el archivo si no existe
        std::cout << "conectado" << std::endl;
    }      // al salir de ámbito, el destructor de Database cierra la conexión

    std::filesystem::remove("tienda.db");      // limpieza, solo para este ejemplo
    return 0;
}
```

Salida:

```text
conectado
```

`":memory:"` en vez de un nombre de archivo abre una base de datos **en memoria**, que desaparece al terminar el programa: muy útil para pruebas.

## Crear una tabla

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iostream>

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);

    db.exec("CREATE TABLE usuario ("
            "id INTEGER PRIMARY KEY AUTOINCREMENT,"
            "nombre TEXT NOT NULL,"
            "email TEXT NOT NULL UNIQUE)");

    std::cout << "tabla creada" << std::endl;
    return 0;
}
```

Salida:

```text
tabla creada
```

`exec` sirve para cualquier sentencia que **no devuelve filas**: `CREATE TABLE`, `INSERT`, `UPDATE`, `DELETE`; si hay un error SQL, lanza `SQLite::Exception` en vez de devolver un código.

## Claves foráneas

SQLite no activa las claves foráneas por defecto: hay que pedirlo:

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iostream>

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("PRAGMA foreign_keys = ON");

    db.exec("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL)");
    db.exec("CREATE TABLE pedido ("
            "id INTEGER PRIMARY KEY,"
            "id_usuario INTEGER NOT NULL,"
            "total REAL NOT NULL,"
            "FOREIGN KEY (id_usuario) REFERENCES usuario(id))");

    db.exec("INSERT INTO usuario (nombre) VALUES ('Ana')");

    bool fallo = false;
    try {
        SQLite::Statement ins(db, "INSERT INTO pedido (id_usuario, total) VALUES (?, ?)");
        ins.bind(1, 99);      // no existe el usuario 99
        ins.bind(2, 10.0);
        ins.exec();
    } catch (const SQLite::Exception&) {
        fallo = true;         // la clave foránea lo impide
    }
    std::cout << std::boolalpha << fallo << std::endl;
    return 0;
}
```

Salida:

```text
true
```

## Insertar con parámetros

Los datos **nunca** se meten en el SQL concatenando texto: se pasan como **parámetros** (`?`), y es la propia librería quien los escapa de forma segura (ver [9.C](t-buenas-practicas.md), inyección SQL).

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iostream>

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, edad INTEGER)");

    SQLite::Statement ins(db, "INSERT INTO usuario (nombre, edad) VALUES (?, ?)");
    ins.bind(1, "Ana"); ins.bind(2, 30); ins.exec();
    ins.reset(); ins.clearBindings();      // la misma Statement, lista para otros valores
    ins.bind(1, "Luis"); ins.bind(2, 25); ins.exec();

    std::cout << "último id: " << db.getLastInsertRowid() << std::endl;
    return 0;
}
```

Salida:

```text
último id: 2
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Construir el `CREATE TABLE` o el `INSERT` concatenando datos del usuario | Usa siempre parámetros (`?` con `bind`) |
| Olvidar `PRAGMA foreign_keys = ON` | SQLite las ignora por defecto: las claves foráneas no se comprueban |
| Abrir una conexión nueva por cada operación | Reutiliza la conexión; ver el pool en [9.C](t-buenas-practicas.md) |
| No capturar `SQLite::Exception` | Un `INSERT` que viola una restricción (`UNIQUE`, `NOT NULL`, una clave foránea) lanza, no devuelve un código silencioso |

## Para practicar

Haz los [ejercicios 9.1 de conexión y tablas](conexion.md).
