# Conexión de JupyterLab y Deepnote con Bases de Datos SQL

Este repositorio contiene dos guías complementarias para trabajar con bases de datos **PostgreSQL** y **SQL Server** desde entornos interactivos de análisis y desarrollo en Python.

---

## 1. [Tutorial: Conectar JupyterLab con PostgreSQL y SQL Server (Windows, cmd)](./Tutorial_Jupyter_Postgres_SQLServer.md)

Guía completa para configurar **JupyterLab** en Windows con dos entornos virtuales independientes (uno para PostgreSQL y otro para SQL Server), habilitando las consultas SQL desde notebooks mediante los comandos mágicos `%sql` y `%%sql`.

**Incluye:**
- Configuración de entornos virtuales (`venv-postgres` y `venv-mssql`)  
- Instalación de dependencias y kernels  
- Conexiones explícitas a PostgreSQL y SQL Server  
- Ejemplos de inserción, actualización y borrado de datos  
- Solución de errores comunes y pruebas con el usuario `estudiante`

**Objetivo:**  
Permitir ejecutar y analizar consultas SQL directamente desde JupyterLab con separación de entornos y sin conflictos de dependencias.

---

## 2. [Tutorial: Conexión de Deepnote con PostgreSQL Local usando Ngrok (Windows)](./Tutorial_Deepnote_Postgres_Windows.md)

Este documento explica cómo conectar una base de datos **PostgreSQL local** en **Windows** con **Deepnote** utilizando **Ngrok** para crear un túnel seguro sin necesidad de abrir puertos.

**Incluye:**
- Creación de cuenta e instalación de Ngrok  
- Configuración del túnel TCP  
- Integración con Deepnote  
- Ejecución de consultas SQL y Python  
- Solución de errores comunes  

**Objetivo:**  
Permitir el acceso remoto seguro a tu base de datos PostgreSQL local desde Deepnote.

---

## Requisitos generales

- **Windows 10 o 11**  
- **Python 3.9+** instalado y agregado al PATH  
- **PostgreSQL** y/o **SQL Server** instalados y en ejecución  
- **Conexión a Internet** para dependencias y Ngrok  

---
