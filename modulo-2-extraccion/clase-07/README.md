# Clase 07 — Python + SQL

> Traer una base de datos entera a un DataFrame con tres líneas


> 🤔 **Para pensar antes de leer:** En SQLBolt escribiste consultas SQL y el sitio te devolvía los resultados en pantalla. Pero esos resultados quedaban ahí — no podías procesarlos, limpiarlos, calcular promedios, ni graficar nada con ellos. ¿Qué necesitarías para que el resultado de una consulta SQL se convierta en un DataFrame de Pandas con el que ya sabes trabajar?


## ¿Qué vamos a ver hoy?

- Qué es SQLite y por qué no requiere instalación
- Conectarse a una base de datos desde Python con `sqlite3`
- Leer el resultado de una consulta SQL directo a un DataFrame con `pd.read_sql()`
- Explorar la estructura de una base de datos desde Python


Hasta ahora trabajaste con dos herramientas por separado. Por un lado, Pandas: carga un CSV, tienes un DataFrame, puedes calcular, filtrar, transformar. Por otro lado, SQL: escribes una consulta, la base de datos te devuelve filas. El problema es que esos dos mundos no hablaban entre sí todavía.

En la práctica, las empresas no guardan sus datos en archivos CSV que alguien actualiza a mano. Los guardan en bases de datos. Entonces el flujo real de trabajo de un analista de datos casi siempre empieza igual: conectarse a una base de datos, hacer una consulta SQL para traer los datos que necesitas, y a partir de ahí trabajar con Python. Esta clase conecta esos dos mundos.

### Qué es SQLite y por qué es el punto de partida ideal

SQLite es un motor de base de datos que tiene una característica que lo diferencia de todos los demás: no necesita un servidor. Una base de datos SQLite es simplemente un archivo en tu computadora, con extensión `.db` o `.sqlite`. Para usarla no tienes que instalar nada, no tienes que configurar un servicio que esté corriendo en segundo plano, no tienes que ingresar credenciales. Simplemente abre el archivo desde Python y empieza a hacer consultas.

Eso la hace perfecta para aprender. El mismo concepto —conectarse, consultar, traer datos— aplica después a PostgreSQL, MySQL, o cualquier otro motor. La diferencia es que en esos casos vas a necesitar un servidor corriendo y credenciales de acceso. Por ahora, con SQLite, esa complejidad no existe y puedes enfocarte en el concepto.

### sqlite3: el módulo que ya viene con Python

Para conectarse a una base de datos SQLite desde Python usas el módulo `sqlite3`. A diferencia de Pandas, que tuviste que instalar con `pip`, `sqlite3` ya viene incluido en Python — no hace falta instalar nada.

```python
import sqlite3
```

Con eso alcanza para tenerlo disponible.

### Crear una base de datos y conectarse

Antes de leer datos, necesitas tener una base de datos a la que conectarte. `sqlite3.connect()` hace dos cosas a la vez: si el archivo que le indicas ya existe, lo abre. Si no existe, lo crea en ese momento.

```python
import sqlite3

conexion = sqlite3.connect("ventas.db")
```

Esa línea crea un archivo llamado `ventas.db` en la misma carpeta donde esté tu script, y abre una conexión a él. `conexion` es el objeto que representa ese canal abierto entre Python y la base de datos — lo vas a necesitar para cualquier operación que hagas a continuación.

### Crear datos de ejemplo para trabajar

Como la base de datos está vacía, antes de hacer consultas necesitas tener algo adentro. Esto no es algo que vayas a hacer en un proyecto real —las bases de datos ya vienen con datos cargados— pero acá te muestra cómo se crea la estructura.

```python
import sqlite3

conexion = sqlite3.connect("ventas.db")
cursor = conexion.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS productos (
        id INTEGER PRIMARY KEY,
        nombre TEXT,
        categoria TEXT,
        precio REAL,
        stock INTEGER
    )
""")

cursor.executemany("""
    INSERT INTO productos (nombre, categoria, precio, stock)
    VALUES (?, ?, ?, ?)
""", [
    ("Notebook",    "tech",    320000, 5),
    ("Mouse",       "tech",    8500,   42),
    ("Escritorio",  "muebles", 85000,  3),
    ("Silla",       "muebles", 45000,  8),
    ("Monitor",     "tech",    210000, 0),
    ("Auriculares", "tech",    32000,  18),
    ("Lámpara",     "muebles", 12000,  25),
])

conexion.commit()
```

Hay dos cosas nuevas acá que vale la pena entender:

`cursor` es el objeto que ejecuta las consultas. La conexión es el canal a la base de datos; el cursor es el que "escribe y lee" dentro de ese canal. La distinción es técnica y por ahora no cambia nada en la práctica — simplemente, para ejecutar SQL desde Python, siempre necesitas un cursor además de la conexión.

`conexion.commit()` confirma los cambios. SQLite no guarda nada de forma permanente hasta que haces `commit()`. Mientras no lo llames, los datos existen solo en memoria, no en el archivo `.db`. Si cierras Python sin hacer `commit()`, los cambios se pierden.

### Leer datos con pd.read_sql()

Ahora viene la parte central de esta clase. `pd.read_sql()` es la función de Pandas que ejecuta una consulta SQL y devuelve el resultado directamente como un DataFrame. Necesita dos cosas: la consulta SQL que quieres ejecutar, y la conexión a la base de datos.

```python
import sqlite3
import pandas as pd

conexion = sqlite3.connect("ventas.db")

df = pd.read_sql("SELECT * FROM productos", conexion)

print(df)
```

```
   id       nombre categoria   precio  stock
0   1     Notebook      tech  320000.0      5
1   2        Mouse      tech    8500.0     42
2   3   Escritorio   muebles   85000.0      3
3   4        Silla   muebles   45000.0      8
4   5      Monitor      tech  210000.0      0
5   6  Auriculares      tech   32000.0     18
6   7      Lámpara   muebles   12000.0     25
```

El resultado es un DataFrame exactamente igual a los que trabajaste en las clases anteriores. A partir de este punto, ya no estás trabajando con SQL — estás trabajando con Pandas como siempre. Puedes seleccionar columnas, filtrar, calcular promedios, todo lo que ya sabes hacer.

Fíjate lo que acaba de pasar: con tres líneas — `import sqlite3`, `sqlite3.connect()`, y `pd.read_sql()` — pasaste de una base de datos a un DataFrame listo para analizar.

### Cualquier consulta SQL funciona

Lo que le pasas a `pd.read_sql()` es SQL normal — el mismo SQL que escribiste en SQLBolt. Puedes usar `WHERE`, `ORDER BY`, `JOIN`, cualquier cosa que ya conozcas:

```python
# Solo los productos de la categoría tech
df_tech = pd.read_sql(
    "SELECT * FROM productos WHERE categoria = 'tech'",
    conexion
)

# Solo los productos con stock disponible, ordenados por precio
df_disponibles = pd.read_sql(
    "SELECT nombre, precio, stock FROM productos WHERE stock > 0 ORDER BY precio DESC",
    conexion
)

print(df_tech)
print(df_disponibles)
```

Cada llamada a `pd.read_sql()` ejecuta la consulta en ese momento y te devuelve los resultados como un DataFrame nuevo. Puedes hacer tantas consultas como necesites sobre la misma conexión.

### Explorar la estructura de una base de datos

Cuando te conectas a una base de datos que ya existe —por ejemplo, una de un proyecto real— lo primero que quieres saber es qué tablas tiene. En SQLite, esa información está guardada en una tabla interna del sistema llamada `sqlite_master`:

```python
tablas = pd.read_sql(
    "SELECT name FROM sqlite_master WHERE type='table'",
    conexion
)

print(tablas)
```

```
        name
0  productos
```

Y para ver la estructura de una tabla en particular —qué columnas tiene y de qué tipo son— usas `PRAGMA table_info()`, que es una instrucción específica de SQLite:

```python
estructura = pd.read_sql(
    "PRAGMA table_info(productos)",
    conexion
)

print(estructura)
```

```
   cid      name     type  notnull dflt_value  pk
0    0        id  INTEGER        0       None   1
1    1    nombre     TEXT        0       None   0
2    2 categoria     TEXT        0       None   0
3    3    precio     REAL        0       None   0
4    4     stock  INTEGER        0       None   0
```

Esto es el equivalente a `.dtypes` en Pandas, pero sobre la base de datos directamente. Te dice el nombre de cada columna, su tipo de dato en SQL, y si es clave primaria.

### Cerrar la conexión cuando terminaste

Una conexión a la base de datos es un recurso que queda abierto mientras tu script corre. Cuando terminaste de trabajar, conviene cerrarla explícitamente:

```python
conexion.close()
```

En scripts cortos que terminan solos, Python cierra la conexión automáticamente al final. Pero es una buena práctica cerrarla de forma explícita — en aplicaciones más largas, dejar conexiones abiertas sin necesidad puede causar problemas.

### Todo junto

Este es el flujo completo de principio a fin: conectarse, leer, analizar:

```python
import sqlite3
import pandas as pd

# 1. Conectarse a la base de datos
conexion = sqlite3.connect("ventas.db")

# 2. Traer los datos que necesitas
df = pd.read_sql(
    "SELECT nombre, categoria, precio, stock FROM productos WHERE stock > 0",
    conexion
)

# 3. A partir de acá, es Pandas de toda la vida
print(f"Productos disponibles: {len(df)}")
print(f"Precio más alto: ${df['precio'].max():,.0f}")
print(f"Precio más bajo: ${df['precio'].min():,.0f}")
print(f"Stock total: {df['stock'].sum()} unidades")
 
# Ver las primeras filas y seleccionar columnas específicas
print(df.head(3))
print(df[["nombre", "precio"]])
 
# 4. Cerrar la conexión cuando terminaste
conexion.close()
```

```
Productos disponibles: 6
Precio más alto: $320,000
Precio más bajo: $8,500
Stock total: 96 unidades
 
  nombre categoria   precio  stock
0   Notebook      tech  320000.0      5
1      Mouse      tech    8500.0     42
2  Escritorio   muebles   85000.0      3
 
        nombre   precio
0     Notebook  320000.0
1        Mouse    8500.0
2   Escritorio   85000.0
3        Silla   45000.0
4      Monitor  210000.0
5  Auriculares   32000.0
```

En cuatro pasos —conectar, consultar, analizar, cerrar— pasaste de una base de datos a un análisis real. El mismo flujo se aplica después con PostgreSQL, cambiando solo la forma de crear la conexión: en lugar de `sqlite3.connect("archivo.db")`, usas una librería como `psycopg2` o `sqlalchemy` con las credenciales del servidor. La llamada a `pd.read_sql()` y todo lo que viene después queda exactamente igual.


## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `sqlite3` | Módulo incluido en Python para conectarse a bases de datos SQLite |
| `sqlite3.connect("archivo.db")` | Abrir (o crear) una base de datos SQLite |
| `conexion.cursor()` | Crear un cursor para ejecutar consultas manualmente |
| `cursor.execute(sql)` | Ejecutar una instrucción SQL (CREATE, INSERT, etc.) |
| `conexion.commit()` | Confirmar los cambios para que queden guardados en el archivo |
| `pd.read_sql(sql, conexion)` | Ejecutar una consulta SQL y traer el resultado como DataFrame |
| `sqlite_master` | Tabla interna de SQLite con la lista de tablas de la base de datos |
| `PRAGMA table_info(tabla)` | Ver la estructura de columnas de una tabla en SQLite |
| `conexion.close()` | Cerrar la conexión cuando ya no la necesitas |


## Recursos adicionales

- [Python Docs — sqlite3](https://docs.python.org/3/library/sqlite3.html)
- [Pandas Docs — pd.read_sql](https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html)
- [Real Python — SQLite and Python](https://realpython.com/python-sqlite-sqlalchemy/)


## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 06 — Introducción a SQL](../clase-06/README.md) · [Módulo 2](../README.md) · Clase 08 — Limpieza de Datos con Pandas →*