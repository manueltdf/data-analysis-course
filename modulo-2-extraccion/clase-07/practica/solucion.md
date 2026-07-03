# Solución — Clase 07: Python + SQL

> ⚠️ **Intentá resolver los ejercicios antes de mirar esto.**

## Ejercicio 1 — ¿Qué hay adentro?  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
# Qué tablas tiene la base de datos
tablas = pd.read_sql(
    "SELECT name FROM sqlite_master WHERE type='table'",
    conexion
)
print("Tablas:")
print(tablas)
 
# Estructura de cada tabla
for tabla in tablas["name"]:
    print(f"\nEstructura de '{tabla}':")
    estructura = pd.read_sql(f"PRAGMA table_info({tabla})", conexion)
    print(estructura[["name", "type"]])
 
conexion.close()
```
 
```
Tablas:
      name
0   duenos
1  mascotas
2  consultas
 
Estructura de 'duenos':
      name     type
0       id  INTEGER
1   nombre     TEXT
2   ciudad     TEXT
3  telefono     TEXT
 
Estructura de 'mascotas':
      name     type
0       id  INTEGER
1   nombre     TEXT
2  especie     TEXT
3     raza     TEXT
4     edad  INTEGER
5 dueno_id  INTEGER
 
Estructura de 'consultas':
         name     type
0          id  INTEGER
1  mascota_id  INTEGER
2       fecha     TEXT
3      motivo     TEXT
4       costo     REAL
5  veterinario     TEXT
```
 
**Por qué importa este ejercicio:** en un proyecto real nunca tenés el script de creación a mano. La combinación de `sqlite_master` para listar tablas y `PRAGMA table_info()` para ver columnas es el primer paso de cualquier exploración de base de datos desconocida — antes de escribir una sola consulta de análisis.
 
 
## Ejercicio 2 — Investigación: ¿cuántas filas tiene cada tabla?  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
tablas = ["duenos", "mascotas", "consultas"]
 
for tabla in tablas:
    resultado = pd.read_sql(f"SELECT COUNT(*) AS total FROM {tabla}", conexion)
    print(f"{tabla}: {resultado['total'][0]} filas")
 
conexion.close()
```
 
```
duenos: 5 filas
mascotas: 10 filas
consultas: 15 filas
```
 
**La respuesta a la pregunta de reflexión:** `COUNT(*)` cuenta las filas directamente en la base de datos, sin traer ningún dato a Python. Si la tabla tiene 10 millones de filas y solo querés saber cuántas hay, traerlas todas con `SELECT *` para después hacer `len(df)` transferiría millones de filas por la red innecesariamente. `COUNT(*)` devuelve un solo número. En tablas pequeñas la diferencia es imperceptible; en producción puede ser la diferencia entre una consulta que tarda milisegundos y una que tarda minutos.
 

 
## Ejercicio 3 — Tu primera consulta desde Python  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_mascotas = pd.read_sql("SELECT * FROM mascotas", conexion)
 
conexion.close()
 
# 1. Cuántas mascotas hay
print(f"Total de mascotas: {len(df_mascotas)}")
 
# 2. Nombres de columnas
print(f"Columnas: {list(df_mascotas.columns)}")
 
# 3. Primeras 3 filas
print(df_mascotas.head(3))
```
 
```
Total de mascotas: 10
Columnas: ['id', 'nombre', 'especie', 'raza', 'edad', 'dueno_id']
 
   id  nombre especie             raza  edad  dueno_id
0   1    Toby   perro         Labrador     4         1
1   2   Mishi    gato           Siamés     2         1
2   3   Rocky   perro  Pastor Alemán     7         2
```
 
 
## Ejercicio 4 — WHERE desde Python  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_perros = pd.read_sql(
    "SELECT * FROM mascotas WHERE especie = 'perro'",
    conexion
)
 
conexion.close()
 
print(f"Cantidad de perros: {len(df_perros)}")
razas = [df_perros["raza"].iloc[i] for i in range(len(df_perros))]
print(f"Razas: {razas}")
```
 
```
Cantidad de perros: 6
Razas: ['Labrador', 'Pastor Alemán', 'Golden Retriever', 'Beagle', 'Bulldog', 'Beagle']
```
 
**La respuesta a la pregunta de reflexión:** filtrar con `WHERE` en la consulta SQL significa que solo viajan por la red (o por la memoria) las filas que cumplían la condición. Si la tabla tiene un millón de filas y solo 50 son perros, `SELECT * FROM mascotas WHERE especie = 'perro'` transfiere 50 filas. `SELECT * FROM mascotas` sin filtro transfiere un millón de filas para que después Python descarte las 999.950 que no necesitabas. En producción, filtrar en SQL siempre es más eficiente que traer todo y filtrar después en Pandas.
 

 
## Ejercicio 5 — Seleccionar columnas específicas  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_consultas = pd.read_sql(
    "SELECT fecha, motivo, costo FROM consultas",
    conexion
)
 
conexion.close()
 
print(f"Costo máximo: ${df_consultas['costo'].max():,.0f}")
print(f"Costo mínimo: ${df_consultas['costo'].min():,.0f}")
print(df_consultas.head())
```
 
```
Costo máximo: $45,000
Costo mínimo: $2,800
 
         fecha             motivo     costo
0   2024-03-01          vacunación    3500.0
1   2024-06-15       control anual    4200.0
2   2024-01-20       esterilización  18000.0
3   2024-02-10      cirugía rodilla  45000.0
4   2024-02-25     control post-op   3800.0
```
 

 
## Ejercicio 6 — LIMIT para explorar  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_primeras = pd.read_sql(
    "SELECT * FROM consultas ORDER BY fecha ASC LIMIT 5",
    conexion
)
 
conexion.close()
 
print(df_primeras)
```
 
```
   id  mascota_id        fecha              motivo     costo  veterinario
0   9           6  2024-01-08       chequeo general  5500.0    Dr. Vera
1   3           2  2024-01-20       esterilización  18000.0   Dra. Sosa
2   4           3  2024-02-10      cirugía rodilla  45000.0    Dr. Vera
3   5           3  2024-02-25     control post-op   3800.0    Dr. Vera
4  12           8  2024-02-28      chequeo general   4200.0  Dra. Romero
```
 

 
## Ejercicio 7 — JOIN desde Python  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_mascotas_duenos = pd.read_sql(
    """
    SELECT mascotas.nombre AS mascota,
           mascotas.especie,
           duenos.nombre AS dueno
    FROM mascotas
    JOIN duenos ON mascotas.dueno_id = duenos.id
    """,
    conexion
)
 
conexion.close()
 
print(df_mascotas_duenos)
```
 
```
       mascota especie           dueno
0         Toby   perro       Ana Torres
1        Mishi    gato       Ana Torres
2        Rocky   perro       Luis Gómez
3         Luna    gato       Clara Díaz
4          Max   perro       Clara Díaz
5         Coco    loro       Pedro Ruiz
6         Nala   perro       Pedro Ruiz
7        Simba    gato      María López
8         Bono   perro      María López
9         Kira    gato       Luis Gómez
```
 
**Nota:** usamos alias (`AS mascota`, `AS dueno`) para que las dos columnas `nombre` —una de `mascotas` y otra de `duenos`— no colisionen en el resultado con el mismo nombre. Sin los alias, una de las dos quedaría sin nombre o pisaría a la otra dependiendo del motor.
 

 
## Ejercicio 8 — Dos JOINs encadenados  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_historial = pd.read_sql(
    """
    SELECT duenos.nombre  AS dueno,
           mascotas.nombre AS mascota,
           consultas.fecha,
           consultas.motivo,
           consultas.costo
    FROM consultas
    JOIN mascotas ON consultas.mascota_id = mascotas.id
    JOIN duenos   ON mascotas.dueno_id    = duenos.id
    ORDER BY consultas.fecha ASC
    """,
    conexion
)
 
conexion.close()
 
print(df_historial)
```
 
```
             dueno  mascota        fecha              motivo     costo
0       Pedro Ruiz     Coco  2024-01-08      chequeo general  5500.0
1       Ana Torres    Mishi  2024-01-20       esterilización  18000.0
2       Luis Gómez    Rocky  2024-02-10     cirugía rodilla  45000.0
3       Luis Gómez    Rocky  2024-02-25     control post-op   3800.0
4      María López    Simba  2024-02-28      chequeo general   4200.0
...
```
 
**La lógica de los dos JOINs:** para llegar desde `consultas` hasta `duenos` no hay una relación directa — `consultas` tiene `mascota_id`, no `dueno_id`. Hay que pasar por `mascotas` como tabla intermedia. Cada `JOIN` agrega una tabla y su condición de conexión; el orden refleja el camino que hay que recorrer entre las tablas.
 

 
## Ejercicio 9 — Investigación: alias en SQL  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_historial = pd.read_sql(
    """
    SELECT d.nombre  AS dueno,
           m.nombre  AS mascota,
           c.fecha,
           c.motivo,
           c.costo
    FROM consultas AS c
    JOIN mascotas  AS m ON c.mascota_id = m.id
    JOIN duenos    AS d ON m.dueno_id   = d.id
    ORDER BY c.fecha ASC
    """,
    conexion
)
 
conexion.close()
 
print(list(df_historial.columns))
print(df_historial.head())
```
 
```
['dueno', 'mascota', 'fecha', 'motivo', 'costo']
```
 
**Por qué los alias de tabla importan:** cuando la consulta tiene dos o tres tablas, escribir `consultas.mascota_id` en lugar de `c.mascota_id` en cada referencia hace el SQL más largo sin agregar claridad. Los alias cortos (`c`, `m`, `d`) hacen la consulta más compacta y fácil de leer de un vistazo, especialmente cuando la misma tabla aparece varias veces en distintas condiciones.
 

 
## Ejercicio 10 — Filtrar después de unir  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
df_sosa = pd.read_sql(
    """
    SELECT m.nombre AS mascota,
           c.fecha,
           c.motivo,
           c.costo
    FROM consultas AS c
    JOIN mascotas  AS m ON c.mascota_id = m.id
    WHERE c.veterinario = 'Dra. Sosa'
    ORDER BY c.fecha DESC
    """,
    conexion
)
 
conexion.close()
 
print(df_sosa)
print(f"\nTotal facturado por Dra. Sosa: ${df_sosa['costo'].sum():,.0f}")
```
 
```
  mascota        fecha           motivo    costo
0    Nala  2024-07-14        vacunación   3500.0
1    Toby  2024-06-15      control anual  4200.0
2    Nala  2024-04-30      control anual  4200.0
3    Kira  2024-03-25     esterilización 15000.0
4    Toby  2024-03-01         vacunación  3500.0
5    Mishi 2024-01-20     esterilización 18000.0
6    Luna  2024-04-05         vacunación  3200.0
 
Total facturado por Dra. Sosa: $51,600
```
 
 
## Ejercicio 11 — El código que no cierra la conexión  
 
**Problema 1 — conexión sin cerrar:**
 
```python
import sqlite3
import pandas as pd
 
def obtener_mascotas(especie, conexion):
    # La conexión se pasa como parámetro en lugar de crearse adentro
    df = pd.read_sql(
        "SELECT * FROM mascotas WHERE especie = ?",
        conexion,
        params=(especie,)
    )
    return df
 
conexion = sqlite3.connect("vetcare.db")
 
perros = obtener_mascotas("perro", conexion)
gatos  = obtener_mascotas("gato",  conexion)
loros  = obtener_mascotas("loro",  conexion)
 
conexion.close()
 
print(perros)
print(gatos)
print(loros)
```
 
**Problema 2 — inyección SQL:**
 
El código original construía la consulta con un f-string:
 
```python
f"SELECT * FROM mascotas WHERE especie = '{especie}'"
```
 
Si alguien llama `obtener_mascotas("perro' OR '1'='1")`, la consulta resultante es:
 
```sql
SELECT * FROM mascotas WHERE especie = 'perro' OR '1'='1'
```
 
`'1'='1'` siempre es verdadero, así que esta consulta devuelve *todas* las filas de la tabla, sin importar la especie. En un sistema real, alguien podría usar esta técnica para acceder a datos que no debería ver, modificar registros, o incluso borrar tablas enteras.
 
La solución es usar parámetros posicionales con `?` en lugar de f-strings. `pd.read_sql()` acepta un argumento `params` que pasa los valores de forma segura:
 
```python
pd.read_sql(
    "SELECT * FROM mascotas WHERE especie = ?",
    conexion,
    params=(especie,)  # la coma al final es necesaria: tiene que ser una tupla
)
```
 
Con esto, el valor de `especie` nunca se interpreta como SQL — siempre se trata como un dato, sin importar qué tenga adentro.
 
 
## Ejercicio 12 — Consultas condicionales con Python  
 
```python
import sqlite3
import pandas as pd
 
def buscar_mascotas(especie=None, edad_maxima=None, ciudad=None):
    conexion = sqlite3.connect("vetcare.db")
 
    # Si se pide filtrar por ciudad, necesitamos JOIN con duenos
    necesita_join = ciudad is not None
 
    if necesita_join:
        base = "SELECT mascotas.* FROM mascotas JOIN duenos ON mascotas.dueno_id = duenos.id"
    else:
        base = "SELECT * FROM mascotas"
 
    condiciones = []
    parametros  = []
 
    if especie is not None:
        condiciones.append("mascotas.especie = ?" if necesita_join else "especie = ?")
        parametros.append(especie)
 
    if edad_maxima is not None:
        condiciones.append("mascotas.edad <= ?" if necesita_join else "edad <= ?")
        parametros.append(edad_maxima)
 
    if ciudad is not None:
        condiciones.append("duenos.ciudad = ?")
        parametros.append(ciudad)
 
    if condiciones:
        query = base + " WHERE " + " AND ".join(condiciones)
    else:
        query = base
 
    df = pd.read_sql(query, conexion, params=parametros)
    conexion.close()
    return df
 
 
print(buscar_mascotas(especie="perro"))
print(buscar_mascotas(edad_maxima=3))
print(buscar_mascotas(especie="gato", ciudad="Buenos Aires"))
print(buscar_mascotas())
```
 
**La lógica central:** la consulta se construye como string acumulando condiciones en una lista. Si la lista queda vacía, no se agrega ningún `WHERE`. Si tiene elementos, se unen con `" AND ".join(condiciones)`. Los valores van separados como parámetros (`params=parametros`) en lugar de interpolados en el string, para evitar la inyección SQL del Ejercicio 11.
 
 
## Ejercicio 13 — Reporte por veterinario  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
df = pd.read_sql("SELECT veterinario, costo FROM consultas", conexion)
conexion.close()
 
# Acumuladores por veterinario — sin groupby()
reporte = {}
 
for i in range(len(df)):
    vet   = df["veterinario"].iloc[i]
    costo = df["costo"].iloc[i]
 
    if vet not in reporte:
        reporte[vet] = {"consultas": 0, "total": 0.0}
 
    reporte[vet]["consultas"] += 1
    reporte[vet]["total"]     += costo
 
# Reporte formateado
print("REPORTE POR VETERINARIO")
print("─" * 40)
for vet, datos in sorted(reporte.items()):
    print(f"{vet}: {datos['consultas']} consultas — ${datos['total']:,.0f}")
```
 
```
REPORTE POR VETERINARIO
────────────────────────────────────────────
Dr. Vera        | 6 consultas |    $61,200
Dra. Romero     | 3 consultas |    $11,200
Dra. Sosa       | 6 consultas |    $51,600
```
 
**Para pensar — la respuesta:** hacen falta ~10 líneas para hacer algo que con `GROUP BY` en SQL (tema de la próxima clase) se escribe en una sola consulta de 3 líneas. El diccionario acumulador, el bucle con `range(len(df))`, la lógica de "inicializar si no existe" — todo eso desaparece.
 
 
## Ejercicio 14 — Múltiples consultas, una conexión  
 
```python
import sqlite3
import pandas as pd
 
conexion = sqlite3.connect("vetcare.db")
 
# 1. Total de dueños
total_duenos = pd.read_sql("SELECT COUNT(*) AS total FROM duenos", conexion)
print(f"Dueños registrados: {total_duenos['total'][0]}")
 
# 2. Distribución por especie
df_especies = pd.read_sql("SELECT especie, COUNT(*) AS cantidad FROM mascotas GROUP BY especie", conexion)
print("\nMascotas por especie:")
print(df_especies)
 
# 3. Consulta más cara y más barata
df_extremos = pd.read_sql(
    """
    SELECT motivo, costo, veterinario
    FROM consultas
    WHERE costo = (SELECT MAX(costo) FROM consultas)
       OR costo = (SELECT MIN(costo) FROM consultas)
    ORDER BY costo DESC
    """,
    conexion
)
print("\nConsulta más cara y más barata:")
print(df_extremos)
 
# 4. Mascotas sin consultas
df_sin_consulta = pd.read_sql(
    """
    SELECT mascotas.nombre, mascotas.especie
    FROM mascotas
    LEFT JOIN consultas ON mascotas.id = consultas.mascota_id
    WHERE consultas.id IS NULL
    """,
    conexion
)
print("\nMascotas sin consultas registradas:")
print(df_sin_consulta)
 
conexion.close()
```
 
```
Dueños registrados: 5
 
Mascotas por especie:
  especie  cantidad
0    gato         4
1    loro         1
2   perro         5
 
Consulta más cara y más barata:
             motivo     costo veterinario
0   cirugía rodilla  45000.0    Dr. Vera
1  desparasitación   2800.0  Dra. Romero
 
Mascotas sin consultas registradas:
  nombre especie
(vacío — todas las mascotas tienen al menos una consulta)
```
 
**Sobre el punto 4:** el patrón `LEFT JOIN ... WHERE columna IS NULL` es exactamente el de la Lección 7 de SQLBolt. Traés todas las mascotas aunque no tengan consultas (`LEFT JOIN`), y después filtrás quedándote solo con las que tienen `NULL` en la columna que vino de `consultas` — es decir, las que no tuvieron coincidencia.
 

← [Volver a ejercicios](./notebook.ipynb)