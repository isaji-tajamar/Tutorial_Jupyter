# Conectar JupyterLab con PostgreSQL y SQL Server en Windows (cmd) usando entornos virtuales (venv) separados y comandos mágicos %sql / %%sql

Este documento explica cómo configurar JupyterLab en Windows para conectarse a dos motores de bases de datos distintos (PostgreSQL y SQL Server) utilizando entornos virtuales independientes y los comandos mágicos `%sql` y `%%sql` de Jupyter.

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
result = %sql SELECT * FROM public.tu_tabla LIMIT 5;
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
SELECT TOP 5 name, object_id, create_date FROM sys.objects ORDER BY create_date DESC;
```

**Obtener resultados como DataFrame**

```python
result = %sql SELECT TOP 10 * FROM dbo.tu_tabla;
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

df = pd.read_sql("SELECT * FROM public.tu_tabla LIMIT 5;", engine)
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

df = pd.read_sql("SELECT TOP 5 * FROM dbo.tu_tabla;", engine)
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
  > Se debe hacer en el entorno virtual correspondiente. Una vez hayas instalado/modificado el kernel debes reiniciarlo en el jupyter notebook. La opción se encuentra en *Kernel > Reiniciar Kernel*.

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
