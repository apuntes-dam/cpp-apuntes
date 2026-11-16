# 9.B Consultas y modificaciones

## `SELECT`: consultar filas

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iomanip>
#include <iostream>

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("CREATE TABLE producto (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, precio REAL NOT NULL)");
    db.exec("INSERT INTO producto (nombre, precio) VALUES ('Teclado', 25.0)");
    db.exec("INSERT INTO producto (nombre, precio) VALUES ('Monitor', 150.0)");

    SQLite::Statement q(db, "SELECT id, nombre, precio FROM producto WHERE precio > ? ORDER BY precio");
    q.bind(1, 50.0);
    while (q.executeStep()) {
        std::cout << q.getColumn(0).getInt() << " " << q.getColumn(1).getText() << " "
                  << std::fixed << std::setprecision(2) << q.getColumn(2).getDouble() << std::endl;
    }
    return 0;
}
```

Salida:

```text
2 Monitor 150.00
```

`executeStep()` avanza una fila cada vez y devuelve `false` cuando ya no quedan: por eso el `while` es la forma natural de recorrer el resultado. `getColumn(i)` da acceso a la columna `i` (desde 0), y cada `get...()` la convierte al tipo pedido.

## `JOIN`: combinar tablas

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iomanip>
#include <iostream>

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL)");
    db.exec("CREATE TABLE pedido (id INTEGER PRIMARY KEY, id_usuario INTEGER NOT NULL, total REAL NOT NULL)");
    db.exec("INSERT INTO usuario (nombre) VALUES ('Ana')");
    db.exec("INSERT INTO pedido (id_usuario, total) VALUES (1, 42.50)");
    db.exec("INSERT INTO pedido (id_usuario, total) VALUES (1, 10.0)");

    SQLite::Statement q(db, "SELECT usuario.nombre, pedido.total FROM pedido "
                            "JOIN usuario ON usuario.id = pedido.id_usuario WHERE usuario.nombre = ?");
    q.bind(1, "Ana");
    while (q.executeStep()) {
        std::cout << q.getColumn(0).getText() << ": " << std::fixed << std::setprecision(2) << q.getColumn(1).getDouble() << std::endl;
    }
    return 0;
}
```

Salida:

```text
Ana: 42.50
Ana: 10.00
```

## `UPDATE` y `DELETE`

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iostream>

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("CREATE TABLE producto (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, precio REAL NOT NULL)");
    db.exec("INSERT INTO producto (nombre, precio) VALUES ('Monitor', 150.0)");

    SQLite::Statement upd(db, "UPDATE producto SET precio = ? WHERE nombre = ?");
    upd.bind(1, 120.0); upd.bind(2, "Monitor");
    int filas = upd.exec();
    std::cout << "filas modificadas: " << filas << std::endl;

    SQLite::Statement del(db, "DELETE FROM producto WHERE nombre = ?");
    del.bind(1, "No existe");
    int borradas = del.exec();
    std::cout << "filas borradas: " << borradas << std::endl;     // 0: no falla, solo no borra nada
    return 0;
}
```

Salida:

```text
filas modificadas: 1
filas borradas: 0
```

`Statement::exec()` devuelve **cuántas filas** se vieron afectadas, no un error si no coincide ninguna: borrar algo que no existe no es un fallo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a `executeStep()` sin comprobar su valor | Para una sola fila esperada, comprueba que devuelve `true` antes de leer columnas |
| Pedir una columna con el tipo equivocado (`getInt()` en un `TEXT`) | SQLite es de tipado dinámico: convierte si puede, pero comprueba el esquema real de la tabla |
| Esperar que `UPDATE`/`DELETE` lancen si no afectan a ninguna fila | No lanzan: comprueba el número que devuelve `exec()` si necesitas saberlo |
| Un `JOIN` sin condición `ON` | Multiplica las filas (producto cartesiano): siempre con su `ON` |

## Para practicar

Haz los [ejercicios 9.2 de consultas y modificaciones](consultas.md).
