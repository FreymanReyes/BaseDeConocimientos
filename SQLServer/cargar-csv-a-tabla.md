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

-- Crea una tabla simple con una sola columna para almacenar los datos importados
CREATE TABLE Comfandi.Radicados (
    Radicado NVARCHAR(255)
);

-- Consulta la ubicación física donde se almacenan los archivos de la base de datos
SELECT physical_name
FROM sys.master_files
WHERE database_id = DB_ID();

-- Carga el archivo CSV a la tabla utilizando BULK INSERT
BULK INSERT Comfandi.Radicados
FROM 'E:\MSSQL\Data\Hoja2.csv'
WITH (
    FIELDTERMINATOR = ',',       -- Delimitador de campo (coma)
    ROWTERMINATOR = '\n',        -- Fin de línea
    FIRSTROW = 2,                -- Empieza desde la fila 2 (para saltar encabezados)
    CODEPAGE = '65001'           -- Para soportar UTF-8 (importante para caracteres especiales)
);
