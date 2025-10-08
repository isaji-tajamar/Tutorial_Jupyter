# Conectar JupyterLab con PostgreSQL y SQL Server en Windows (cmd) usando entornos virtuales (venv) separados y comandos mágicos %sql / %%sql

Este documento explica cómo configurar JupyterLab en Windows para conectarse a dos motores de bases de datos distintos (PostgreSQL y SQL Server) utilizando entornos virtuales independientes y los comandos mágicos `%sql` y `%%sql` de Jupyter.

---
# Índice

## Parte I — Configuración de entornos y conexión

1. [Requisitos previos](#requisitos-previos)  
2. [Configurar entorno virtual para PostgreSQL](#1-configurar-entorno-virtual-para-postgresql)  
3. [Configurar entorno virtual para SQL Server](#2-configurar-entorno-virtual-para-sql-server)  
4. [Abrir JupyterLab y seleccionar kernel](#3-abrir-jupyterlab-y-seleccionar-kernel)  
5. [Conectar y ejecutar consultas con %sql y %%sql](#4-conectar-y-ejecutar-consultas-con-sql-y-sql)  
6. [Cadenas de conexión (resumen)](#5-cadenas-de-conexión-resumen)  
7. [Consultar con SQLAlchemy y pandas (sin %sql)](#6-consultar-con-sqlalchemy-y-pandas-sin-sql)  
8. [Solución de errores comunes](#7-solución-de-errores-comunes)  
9. [Checklist rápido](#8-checklist-rápido)  
10. [Buenas prácticas de seguridad](#9-buenas-prácticas-de-seguridad)  
11. [Resultado final](#10-resultado-final)  

---

## Parte II — Pruebas prácticas de inserción, actualización y borrado

1. [PostgreSQL — conexión y pruebas](#1-postgresql--conexión-y-pruebas)  
2. [SQL Server — conexión y pruebas](#2-sql-server-instancia-nombrada--conexión-y-pruebas)  
3. [Notas y variaciones útiles](#3-notas-y-variaciones-útiles)  
4. [Comprobaciones rápidas de permisos con el usuario estudiante](#4-comprobaciones-rápidas-de-permisos-con-el-usuario-estudiante)


---

## Requisitos previos

- **Python 3.9 o superior** instalado y en el PATH (`python --version`)
- **Permisos administrativos** para instalar software
- **Microsoft ODBC Driver 18 for SQL Server** (solo necesario para SQL Server)

Instalar el Driver ODBC 18 si no está disponible:
[Guía de instalación de Microsoft Learn](https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver17)

Crear la carpeta base de trabajo (puedes establecer la ruta que quieras):

```bat
mkdir C:\data\jupyter-db
cd /d C:\data\jupyter-db
```

---

## 1) Configurar entorno virtual para PostgreSQL

### 1.1 Crear y activar el entorno

```bat
python -m venv venv-postgres
venv-postgres\Scripts\activate.bat
```

### 1.2 Instalar dependencias necesarias

```bat
python -m pip install --upgrade pip
pip install jupyterlab ipykernel jupysql sqlalchemy psycopg2-binary pandas
```

### 1.3 Registrar el kernel en Jupyter

```bat
python -m ipykernel install --user --name "venv-postgres" --display-name "Python (venv-postgres)"
```

---

## 2) Configurar entorno virtual para SQL Server

### 2.1 Crear y activar el entorno

```bat
cd /d C:\data\jupyter-db
python -m venv venv-mssql
venv-mssql\Scripts\activate.bat
```

### 2.2 Instalar dependencias necesarias

```bat
python -m pip install --upgrade pip
pip install jupyterlab ipykernel jupysql sqlalchemy pyodbc pandas
```

### 2.3 Registrar el kernel en Jupyter

```bat
python -m ipykernel install --user --name "venv-mssql" --display-name "Python (venv-mssql)"
```

Verificar que el driver ODBC esté disponible:

```python
import pyodbc
print(pyodbc.drivers())
```

Debe aparecer: `ODBC Driver 18 for SQL Server`.

---

## 3) Abrir JupyterLab y seleccionar kernel

Iniciar JupyterLab desde cualquiera de los entornos:

```bat
cd /d C:\data\jupyter-db
venv-postgres\Scripts\activate.bat
jupyter lab
```

- Notebooks que usen PostgreSQL → kernel **Python (venv-postgres)**
- Notebooks que usen SQL Server → kernel **Python (venv-mssql)**

---

## 4) Conectar y ejecutar consultas con %sql y %%sql

### 4.1 Reglas básicas

- `%sql` se usa para una única línea de consulta SQL
- `%%sql` se usa al comienzo de una celda que contenga solo código SQL
- Las celdas con `%%sql` no deben contener Python

---

### 4.2 Conexión a PostgreSQL

**Celda 1 (Python): cargar extensión y conectar**

```python
%load_ext sql

# Cadena de conexión explícita a PostgreSQL
%sql postgresql+psycopg2://estudiante:1234@localhost:5432/prueba
```

**Celda 2 (SQL puro): ejecutar consulta**

```sql
%%sql
SELECT version();
```

**Obtener resultados como DataFrame**

```python
result = %sql SELECT * FROM tu_tabla LIMIT 5;
df = result.DataFrame()
df.head()
```

---

### 4.3 Conexión a SQL Server

**Celda 1 (Python): cargar extensión y conectar**

```python
%load_ext sql

# Cadena de conexión explícita (SQL Login)
%sql mssql+pyodbc://estudiante:1234@localhost\SQLEXPRESS/prueba?driver=ODBC+Driver+18+for+SQL+Server&Encrypt=yes&TrustServerCertificate=yes


# Alternativa (SQL Login)
# %sql mssql+pyodbc://estudiante:1234@localhost:1433/prueba?driver=ODBC+Driver+18+for+SQL+Server&Encrypt=yes&TrustServerCertificate=yes

# Alternativa (autenticación integrada de Windows)
# %sql mssql+pyodbc://@localhost:1433/prueba?driver=ODBC+Driver+18+for+SQL+Server&Trusted_Connection=yes&Encrypt=yes&TrustServerCertificate=yes
```

**Celda 2 (SQL puro): ejecutar consulta**

```sql
%%sql
SELECT @@VERSION;
```

**Consulta de ejemplo**

```sql
%%sql
SELECT TOP 5 * FROM tu_tabla;
```

**Obtener resultados como DataFrame**

```python
result = %sql SELECT TOP 10 * FROM tu_tabla;
df = result.DataFrame()
df.head()
```

---

## 5) Cadenas de conexión (resumen)

| Motor | Ejemplo de cadena de conexión | Driver |
|--------|--------------------------------|---------|
| PostgreSQL | `postgresql+psycopg2://estudiante:1234@localhost:5432/prueba` | psycopg2-binary |
| SQL Server (SQL Login) | `mssql+pyodbc://estudiante:1234@localhost:1433/prueba?driver=ODBC+Driver+18+for+SQL+Server&Encrypt=yes&TrustServerCertificate=yes` | pyodbc |
| SQL Server (instancia nombrada) | `mssql+pyodbc://estudiante:1234@localhost\SQLEXPRESS/prueba?driver=ODBC+Driver+18+for+SQL+Server&Encrypt=yes&TrustServerCertificate=yes` | pyodbc |
| SQL Server (autenticación de Windows) | `mssql+pyodbc://@localhost:1433/prueba?driver=ODBC+Driver+18+for+SQL+Server&Trusted_Connection=yes&Encrypt=yes&TrustServerCertificate=yes` | pyodbc |

---

## 6) Consultar con SQLAlchemy y pandas (sin %sql)

### PostgreSQL

```python
from sqlalchemy import create_engine, text
import pandas as pd

url = "postgresql+psycopg2://estudiante:1234@localhost:5432/prueba"
engine = create_engine(url, pool_pre_ping=True)

with engine.begin() as conn:
    print(conn.execute(text("SELECT version();")).scalar())

df = pd.read_sql("SELECT * FROM tu_tabla LIMIT 5;", engine)
df.head()
```

### SQL Server

```python
from sqlalchemy import create_engine, text
import pandas as pd

url = "mssql+pyodbc://estudiante:1234@localhost:1433/prueba?driver=ODBC+Driver+18+for+SQL+Server&Encrypt=yes&TrustServerCertificate=yes"
engine = create_engine(url, pool_pre_ping=True)

with engine.begin() as conn:
    print(conn.execute(text("SELECT @@VERSION;")).scalar())

df = pd.read_sql("SELECT TOP 5 * FROM tu_tabla;", engine)
df.head()
```

---

## 7) Solución de errores comunes

- **Error:** `SyntaxError: invalid syntax`  
**Causa:** Python y SQL en la misma celda.  
**Solución:** usa `%%sql` solo en celdas que contengan SQL.

- **Error:** `ModuleNotFoundError: No module named 'psycopg2'`  
**Causa:** kernel incorrecto o falta el driver.  
**Solución:** usa el entorno correcto o instala el paquete (`pip install psycopg2-binary`).

- **Error:** `IM002 ... data source name not found`  
**Causa:** el driver ODBC no está instalado o el nombre no coincide.  
**Solución:** verifica el nombre exacto con `pyodbc.drivers()`.

- **Error:** `KeyError` en prettytable al mostrar resultados  
**Causa:** incompatibilidad de estilos en jupysql / prettytable.  
**Solución:**
  ```bat
  pip install prettytable==3.10.0
  ```

  > [!WARNING]
  > Se debe hacer en el entorno virtual correspondiente. Una vez hayas instalado/modificado el kernel debes reiniciarlo en el jupyter notebook. La opción se encuentra en **Kernel > Reiniciar Kernel**.

---

## 8) Resumen rápido

**PostgreSQL**
```bat
cd /d C:\data\jupyter-db
python -m venv venv-postgres
venv-postgres\Scripts\activate.bat
pip install jupyterlab ipykernel jupysql sqlalchemy psycopg2-binary pandas
python -m ipykernel install --user --name "venv-postgres" --display-name "Python (venv-postgres)"
jupyter lab
```

**SQL Server**
```bat
cd /d C:\data\jupyter-db
python -m venv venv-mssql
venv-mssql\Scripts\activate.bat
pip install jupyterlab ipykernel jupysql sqlalchemy pyodbc pandas
python -m ipykernel install --user --name "venv-mssql" --display-name "Python (venv-mssql)"
jupyter lab
```

---

## 9) Resultado final

Tras seguir estos pasos tendrás:

- Dos entornos virtuales separados:
  ```
  C:\data\jupyter-db\venv-postgres
  C:\data\jupyter-db\venv-mssql
  ```
- Dos kernels registrados en Jupyter:
  - Python (venv-postgres)
  - Python (venv-mssql)
- Capacidad de ejecutar consultas SQL directamente desde notebooks con:
  ```python
  %sql postgresql+psycopg2://...
  %sql mssql+pyodbc://...
  ```
- Resultados mostrados en tablas o DataFrames según tu configuración.


# Pruebas de inserción, actualización y borrado desde Jupyter para PostgreSQL y SQL Server (Windows, cmd)

Este anexo añade **pruebas DML** (INSERT, UPDATE, DELETE) para ambos motores, usando los **comandos mágicos `%sql`/`%%sql`** de Jupyter. 
Las conexiones usan el usuario `estudiante` y password `1234`. Para SQL Server se usa **instancia nombrada** (`localhost\SQLEXPRESS`).

> [!NOTE] 
> **Regla clave:** las celdas con `%%sql` deben contener **solo SQL** (sin Python en la misma celda).

---

## 1) PostgreSQL — conexión y pruebas

### 1.1 Conectar (celda Python)

```python
%load_ext sql

# Cadena de conexión explícita a PostgreSQL
%sql postgresql+psycopg2://estudiante:1234@localhost:5432/prueba
```

### 1.2 Crear tabla de pruebas (celda SQL)

```sql
%%sql
DROP TABLE IF EXISTS ventas_test;

CREATE TABLE ventas_test (
    id          SERIAL PRIMARY KEY,
    producto    TEXT        NOT NULL,
    cantidad    INTEGER     NOT NULL CHECK (cantidad > 0),
    precio      NUMERIC(10,2) NOT NULL CHECK (precio >= 0),
    fecha_venta TIMESTAMP   NOT NULL DEFAULT NOW()
);
```

### 1.3 INSERT: insertar filas de ejemplo (celda SQL)

```sql
%%sql
INSERT INTO ventas_test (producto, cantidad, precio)
VALUES 
  ('Teclado', 2, 19.90),
  ('Ratón',   1, 12.50),
  ('Monitor', 3, 159.00);
```

Verificar:
```sql
%%sql
SELECT * FROM ventas_test ORDER BY id;
```

### 1.4 UPDATE: actualizar registros (celda SQL)

```sql
%%sql
-- Actualizar cantidad del producto 'Teclado' a 3
UPDATE ventas_test
   SET cantidad = 3
 WHERE producto = 'Teclado';

-- Subir precio un 10 % para todos los productos
UPDATE ventas_test
   SET precio = ROUND(precio * 1.10, 2);
```

Verificar:
```sql
%%sql
SELECT id, producto, cantidad, precio FROM ventas_test ORDER BY id;
```

### 1.5 DELETE: borrar filas (celda SQL)

```sql
%%sql
-- Borrar una fila por condición
DELETE FROM ventas_test
 WHERE producto = 'Ratón';

-- Borrar por rango de precio (ejemplo)
DELETE FROM ventas_test
 WHERE precio > 200;
```

Verificar:
```sql
%%sql
SELECT id, producto, cantidad, precio FROM ventas_test ORDER BY id;
```

### 1.6 Limpieza (opcional)

```sql
%%sql
DROP TABLE IF EXISTS ventas_test;
```

---

## 2) SQL Server (instancia nombrada) — conexión y pruebas

### 2.1 Conectar (celda Python)

```python
%load_ext sql

# Cadena de conexión explícita con instancia nombrada (SQLEXPRESS). 
# Nota: en %sql no es necesario escapar la barra inversa de la instancia.
%sql mssql+pyodbc://estudiante:1234@localhost\SQLEXPRESS/prueba?driver=ODBC+Driver+18+for+SQL+Server&Encrypt=yes&TrustServerCertificate=yes
```

### 2.2 Crear tabla de pruebas (celda SQL)

```sql
%%sql
IF OBJECT_ID('ventas_test', 'U') IS NOT NULL
    DROP TABLE ventas_test;

CREATE TABLE ventas_test (
    id          INT IDENTITY(1,1) PRIMARY KEY,
    producto    NVARCHAR(200) NOT NULL,
    cantidad    INT           NOT NULL CHECK (cantidad > 0),
    precio      DECIMAL(10,2) NOT NULL CHECK (precio >= 0),
    fecha_venta DATETIME2     NOT NULL DEFAULT SYSDATETIME()
);
```

### 2.3 INSERT: insertar filas de ejemplo (celda SQL)

```sql
%%sql
INSERT INTO ventas_test (producto, cantidad, precio)
VALUES 
  (N'Teclado', 2, 19.90),
  (N'Ratón',   1, 12.50),
  (N'Monitor', 3, 159.00);
```

Verificar:
```sql
%%sql
SELECT * FROM ventas_test ORDER BY id;
```

### 2.4 UPDATE: actualizar registros (celda SQL)

```sql
%%sql
-- Actualizar cantidad del producto 'Teclado' a 3
UPDATE ventas_test
   SET cantidad = 3
 WHERE producto = N'Teclado';

-- Subir precio un 10 % para todos los productos
UPDATE ventas_test
   SET precio = ROUND(precio * 1.10, 2);
```

Verificar:
```sql
%%sql
SELECT id, producto, cantidad, precio FROM ventas_test ORDER BY id;
```

### 2.5 DELETE: borrar filas (celda SQL)

```sql
%%sql
-- Borrar una fila por condición
DELETE FROM ventas_test
 WHERE producto = N'Ratón';

-- Borrar por rango de precio (ejemplo)
DELETE FROM ventas_test
 WHERE precio > 200;
```

Verificar:
```sql
%%sql
SELECT id, producto, cantidad, precio FROM ventas_test ORDER BY id;
```

### 2.6 Limpieza (opcional)

```sql
%%sql
IF OBJECT_ID('ventas_test', 'U') IS NOT NULL
    DROP TABLE ventas_test;
```

---

## 3) Notas y variaciones útiles

- Si tu SQL Server no se llama `SQLEXPRESS`, sustituye el nombre de la instancia en la cadena de conexión. 
  En entornos corporativos, a veces es más fiable usar un puerto fijo: `localhost:1433` en lugar de la instancia.
- Para PostgreSQL, si usas SSL obligatorio, añade parámetros como `?sslmode=require` a la URL.
- En SQL Server se usa `TOP N`; en PostgreSQL, `LIMIT N`. Ajusta los ejemplos según el motor.
- Si `%%sql` muestra errores de formato, usa DataFrames:
  ```python
  result = %sql SELECT * FROM ventas_test;
  df = result.DataFrame()
  df.head()
  ```

---

## 4) Comprobaciones rápidas de permisos con el usuario estudiante

- Insertar fila mínima válida:
  ```sql
  %%sql
  INSERT INTO ventas_test (producto, cantidad, precio) VALUES ('Cable', 1, 3.50);
  ```
  y
  ```sql
  %%sql
  INSERT INTO ventas_test (producto, cantidad, precio) VALUES (N'Cable', 1, 3.50);
  ```
- Verificar que puedes actualizar y borrar al menos una fila en cada tabla.
- Si recibes errores de permisos, confirma que el usuario `estudiante` tiene privilegios de `INSERT`, `UPDATE` y `DELETE` sobre las tablas creadas.
