# 📥 Cargar archivo CSV a una tabla en SQL Server

Este script permite cargar datos desde un archivo `.csv` directamente a una tabla en SQL Server utilizando `BULK INSERT`.  
Incluye configuraciones previas necesarias para habilitar esta funcionalidad.

---

## 🔧 1. Habilitar opciones avanzadas y consultas distribuidas

```sql
-- Habilita la visualización de opciones avanzadas en SQL Server
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;

-- Habilita las consultas distribuidas ad hoc necesarias para usar BULK INSERT desde archivos
EXEC sp_configure 'Ad Hoc Distributed Queries', 1;
RECONFIGURE;

> **ℹ️ Nota:** Esta tabla que veras a continuaciòn es un ejemplo, la cantidad de columnas depende de la necesidad.

-- Crea una tabla simple con una sola columna para almacenar los datos importados
CREATE TABLE dbo.Radicados (
    Radicado NVARCHAR(255)
);

-- Consulta la ubicación física donde se almacenan los archivos de la base de datos
SELECT physical_name
FROM sys.master_files
WHERE database_id = DB_ID();

> ⚠️ ** Advertencia:** El archivo de prueba contiene una Hoja y una columna la cual se llama Radicado.

-- Carga el archivo CSV a la tabla utilizando BULK INSERT
BULK INSERT dbo.Radicados
FROM 'E:\MSSQL\Data\archivo.csv'
WITH (
    FIELDTERMINATOR = ',',       -- Delimitador de campo (coma)
    ROWTERMINATOR = '\n',        -- Fin de línea
    FIRSTROW = 2,                -- Empieza desde la fila 2 (para saltar encabezados)
    CODEPAGE = '65001'           -- Para soportar UTF-8 (importante para caracteres especiales)
);
