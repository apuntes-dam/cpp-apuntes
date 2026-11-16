# 9.C Buenas prácticas

## Transacciones: todo o nada

Cuando varias operaciones deben completarse **juntas** (insertar un pedido y descontar el stock), una **transacción** asegura que, si algo falla a mitad, **nada** de lo anterior queda guardado:

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iostream>

bool crear_pedido(SQLite::Database& db, int idUsuario, double total, int idProducto, int cantidad) {
    SQLite::Transaction tx(db);      // empieza la transacción

    SQLite::Statement ins(db, "INSERT INTO pedido (id_usuario, total) VALUES (?, ?)");
    ins.bind(1, idUsuario); ins.bind(2, total); ins.exec();

    SQLite::Statement q(db, "SELECT stock FROM producto WHERE id = ?");
    q.bind(1, idProducto);
    q.executeStep();
    int stock = q.getColumn(0).getInt();
    if (stock < cantidad) {
        return false;      // tx se destruye sin commit: deshace sola (rollback)
    }

    SQLite::Statement upd(db, "UPDATE producto SET stock = stock - ? WHERE id = ?");
    upd.bind(1, cantidad); upd.bind(2, idProducto); upd.exec();

    tx.commit();            // confirma: ahora sí se guarda todo
    return true;
}

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("CREATE TABLE pedido (id INTEGER PRIMARY KEY, id_usuario INTEGER, total REAL)");
    db.exec("CREATE TABLE producto (id INTEGER PRIMARY KEY, stock INTEGER)");
    db.exec("INSERT INTO producto (id, stock) VALUES (1, 5)");

    std::cout << std::boolalpha << crear_pedido(db, 1, 20.0, 1, 2) << std::endl;
    std::cout << crear_pedido(db, 1, 20.0, 1, 999) << std::endl;      // no hay stock: se deshace entero

    SQLite::Statement c(db, "SELECT COUNT(*) FROM pedido");
    c.executeStep();
    std::cout << "pedidos guardados: " << c.getColumn(0).getInt() << std::endl;      // solo el primero
    return 0;
}
```

Salida:

```text
true
false
pedidos guardados: 1
```

Si `tx` se destruye (sale de ámbito) **sin** haber llamado a `commit()`, SQLiteCpp la **deshace automáticamente** (*rollback*) en su destructor: no hace falta acordarse de deshacerla a mano en cada camino de error, basta con devolver pronto y dejar que el objeto `tx` se destruya.

## Inyección SQL: por qué nunca se concatena

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iostream>
#include <string>
#include <vector>

std::vector<std::string> buscar_mal(SQLite::Database& db, const std::string& nombre) {
    // NUNCA hagas esto: el texto del usuario se mete directamente en el SQL
    std::string sql = "SELECT nombre FROM usuario WHERE nombre = '" + nombre + "'";
    SQLite::Statement q(db, sql);
    std::vector<std::string> out;
    while (q.executeStep()) out.push_back(q.getColumn(0).getText());
    return out;
}

std::vector<std::string> buscar_bien(SQLite::Database& db, const std::string& nombre) {
    SQLite::Statement q(db, "SELECT nombre FROM usuario WHERE nombre = ?");
    q.bind(1, nombre);
    std::vector<std::string> out;
    while (q.executeStep()) out.push_back(q.getColumn(0).getText());
    return out;
}

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("CREATE TABLE usuario (nombre TEXT)");
    db.exec("INSERT INTO usuario (nombre) VALUES ('Ana')");

    std::string ataque = "' OR '1'='1";      // intenta que la condición sea siempre verdadera
    for (auto& n : buscar_mal(db, ataque)) std::cout << n << " ";
    std::cout << std::endl;                  // ¡devuelve a Ana, sin que el nombre coincida!

    for (auto& n : buscar_bien(db, ataque)) std::cout << n << " ";
    std::cout << "(vacío)" << std::endl;      // trata el texto como un dato, no como SQL
    return 0;
}
```

Salida:

```text
Ana 
(vacío)
```

Con parámetros, el texto del usuario **nunca** se interpreta como código SQL, pase lo que pase dentro de él.

## Un pool: reutilizar conexiones

Abrir una conexión no es gratis. En una aplicación que atiende muchas peticiones a la vez, conviene un **pool**: un grupo de conexiones ya abiertas, que se piden prestadas y se devuelven. SQLiteCpp no trae uno propio; la idea, con una sola conexión compartida de forma simple:

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <iostream>

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);   // en una app real, vendría de un pool
    db.exec("CREATE TABLE visita (id INTEGER PRIMARY KEY)");

    for (int i = 0; i < 3; i++) {
        db.exec("INSERT INTO visita DEFAULT VALUES");      // reutiliza la MISMA conexión
    }
    SQLite::Statement q(db, "SELECT COUNT(*) FROM visita");
    q.executeStep();
    std::cout << q.getColumn(0).getInt() << std::endl;
    return 0;
}
```

Salida:

```text
3
```

## El patrón DAO: una capa para acceder a los datos

**DAO** (*Data Access Object*) agrupa toda la lógica de una tabla en un sitio, para que el resto del programa no escriba SQL directamente:

Instalado con vcpkg: `SQLiteCpp`.

```cpp
#include <SQLiteCpp/SQLiteCpp.h>
#include <cstdint>
#include <iostream>
#include <string>
#include <vector>

struct Usuario {
    int64_t id;
    std::string nombre;
};

class UsuarioDao {
    SQLite::Database& db;
public:
    explicit UsuarioDao(SQLite::Database& db) : db(db) {}

    int64_t crear(const std::string& nombre) {
        SQLite::Statement ins(db, "INSERT INTO usuario (nombre) VALUES (?)");
        ins.bind(1, nombre);
        ins.exec();
        return db.getLastInsertRowid();
    }

    Usuario buscarPorId(int64_t id) {
        SQLite::Statement q(db, "SELECT id, nombre FROM usuario WHERE id = ?");
        q.bind(1, id);
        q.executeStep();
        return Usuario{q.getColumn(0).getInt64(), q.getColumn(1).getText()};
    }

    std::vector<Usuario> listar() {
        SQLite::Statement q(db, "SELECT id, nombre FROM usuario");
        std::vector<Usuario> out;
        while (q.executeStep()) out.push_back(Usuario{q.getColumn(0).getInt64(), q.getColumn(1).getText()});
        return out;
    }
};

int main() {
    SQLite::Database db(":memory:", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
    db.exec("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT)");
    UsuarioDao dao(db);

    dao.crear("Ana");
    dao.crear("Luis");

    Usuario ana = dao.buscarPorId(1);
    std::cout << ana.id << " " << ana.nombre << std::endl;
    for (auto& u : dao.listar()) std::cout << u.id << ": " << u.nombre << std::endl;
    return 0;
}
```

Salida:

```text
1 Ana
1: Ana
2: Luis
```

El resto del programa llama a `dao.crear(...)` o `dao.listar()`; si mañana cambias de SQLite a otro motor, solo tocas `UsuarioDao`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Hacer varias operaciones relacionadas sin transacción | Si una falla a mitad, los datos quedan a medias; usa `SQLite::Transaction` |
| Concatenar texto del usuario en el SQL | Siempre parámetros (`?` con `bind`), nunca construir el texto con datos externos |
| Abrir y cerrar una conexión por cada operación en una app con mucho tráfico | Usa un pool en vez de abrir una conexión nueva cada vez |
| Esparcir sentencias SQL por todo el programa | Agrúpalas en una capa DAO, una por tabla |

## Para practicar

Haz los [ejercicios 9.3 del pool, la seguridad, las transacciones y el DAO](buenas-practicas.md).
