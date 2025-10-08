# Tutorial: Conexión de Deepnote con PostgreSQL Local usando Ngrok (Windows)

Este documento explica paso a paso cómo conectar una base de datos **PostgreSQL local** en **Windows** con **Deepnote** usando **Ngrok**.  
Ngrok permite crear un túnel seguro a tu máquina local sin necesidad de abrir puertos.

---

## Índice

1. [Requisitos previos](#1-requisitos-previos)
2. [Crear una cuenta en Ngrok](#2-crear-una-cuenta-en-ngrok)
3. [Instalar Ngrok en Windows](#3-instalar-ngrok-en-windows)
4. [Ejecutar el túnel con Ngrok](#4-ejecutar-el-túnel-con-ngrok)
5. [Crear la cuenta y el workspace en Deepnote](#5-crear-la-cuenta-y-el-workspace-en-deepnote)
6. [Crear el proyecto e integrar PostgreSQL](#6-crear-el-proyecto-e-integrar-postgresql)
7. [Probar consultas SQL y Python](#7-probar-consultas-sql-y-python)
8. [Solución de problemas](#8-solución-de-problemas)

---

## 1. Requisitos previos

Antes de comenzar, asegúrate de tener:

- **PostgreSQL** instalado y ejecutándose en tu equipo local  
  (por defecto escucha en `localhost:5432`)
- **Una cuenta gratuita en Deepnote**  
  [https://deepnote.com](https://deepnote.com)
- **Una cuenta gratuita en Ngrok**  
  [https://ngrok.com](https://ngrok.com)

---

## 2. Crear una cuenta en Ngrok

1. Accede a la página de registro:  
   [https://ngrok.com/signup](https://ngrok.com/signup)
2. Completa el formulario con tus datos:
   - Nombre
   - Correo electrónico
   - Contraseña
3. O inicia sesión con **GitHub** o **Google**.

**Captura de ejemplo:**  
![Crear cuenta en Ngrok](./img/deepnote/captura8_cuentangrok.png)

---

## 3. Instalar Ngrok en Windows

1. En tu panel de Ngrok, entra en la sección **Setup & Installation**.  
2. Elige la opción **Microsoft Store Installer**.

**Captura de ejemplo:**  
![Instalar Ngrok desde Microsoft Store](./img/deepnote/captura9_instalar_ngrok.png)

3. Una vez instalado, abre **PowerShell** y ejecuta:
   ```bash
   ngrok config add-authtoken TU_TOKEN_AQUI
   ```
   (puedes copiar tu token desde el panel de usuario de Ngrok)

---

## 4. Ejecutar el túnel con Ngrok

1. En PowerShell, lanza el túnel hacia tu PostgreSQL local:
   ```bash
   ngrok tcp 5432
   ```
2. Verás una salida similar a:

   ```
   Session Status                online
   Forwarding                    tcp://2.tcp.eu.ngrok.io:16276 -> localhost:5432
   ```

   - **Host:** `2.tcp.eu.ngrok.io`
   - **Port:** `16276`

**Captura de ejemplo:**  
![Ngrok ejecutándose](./img/deepnote/captura10_ngrok_running.png)

Mantén esta ventana abierta mientras trabajas con Deepnote.

---

## 5. Crear la cuenta y el workspace en Deepnote

1. Accede a [https://deepnote.com](https://deepnote.com) y haz clic en **Get started**.  
   ![Página principal de Deepnote](./img/deepnote/captura1_gettingstarted.png)

2. Completa tus datos personales y elige tu rol.  
   ![Formulario de registro en Deepnote](./img/deepnote/captura2_crear_cuenta.png)

3. Crea un **Workspace** con el nombre que prefieras.  
   Ejemplo: `IABigData`  
   ![Creación del workspace](./img/deepnote/captura3_crear_workspace.png)

---

## 6. Crear el proyecto e integrar PostgreSQL

1. En el panel izquierdo, selecciona **+ New project**.  
   ![Crear nuevo proyecto](./img/deepnote/captura4_new_project.png)

2. Dentro del proyecto, selecciona **Connect or add a new integration**.  
   ![Agregar integración en Deepnote](./img/deepnote/captura5_add_integration.png)

3. Elige **PostgreSQL** en la lista.  
   ![Seleccionar PostgreSQL](./img/deepnote/captura6_select_postgres.png)

4. Rellena los datos de conexión con los valores que muestra Ngrok:

   | Campo | Valor |
   |--------|--------|
   | Integration name | postgres |
   | Host name | `host de ngrok` |
   | Port | `puerto de ngrok` |
   | Database | Tu bbdd |
   | Username | Tu usuario |
   | Password | Tu contraseña |
   | Enforce SSL encryption | Desactivado (para pruebas) |

   ![Configuración de integración PostgreSQL](./img/deepnote/captura7_configurar_integration.png)

5. Pulsa **Create integration** y espera la confirmación de conexión.

---

## 7. Probar consultas SQL y Python

### 7.1 Usando celdas SQL
1. Crea una nueva celda en el notebook.
2. Cambia el tipo de celda a **SQL**.
3. Selecciona la conexión creada y escribe tus consultas:

```sql
CREATE TABLE alumnos (
  id SERIAL PRIMARY KEY,
  nombre TEXT NOT NULL,
  nota NUMERIC
);

INSERT INTO alumnos (nombre, nota)
VALUES ('Ana', 8.5), ('Luis', 7.2);

SELECT * FROM alumnos;
```

---

### 7.2 Usando Python y pandas

```python
from sqlalchemy import create_engine, text
import pandas as pd

DB_URL = "postgresql+psycopg2://estudiante:1234@2.tcp.eu.ngrok.io:16276/prueba"
engine = create_engine(DB_URL)

df = pd.read_sql("SELECT * FROM alumnos;", engine)
df.head()
```

---

## 8. Solución de problemas

| Problema | Causa | Solución |
|-----------|--------|-----------|
| Error de conexión | Ngrok cerrado o puerto distinto | Repite `ngrok tcp 5432` y actualiza el puerto en Deepnote |
| Error SSL | PostgreSQL sin certificado | Desactiva "Enforce SSL encryption" |
| Acceso denegado | Usuario/contraseña incorrectos | Verifica credenciales locales |
| Conexión interrumpida | Se cerró la consola de Ngrok | Mantén Ngrok abierto mientras trabajas |

---
