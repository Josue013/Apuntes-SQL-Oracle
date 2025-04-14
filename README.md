# Apuntes Curso Oracle

## Tablas de datos

### 1. Tipos de datos

Los tipos de datos caracteres:

```sql
CHAR       -- Longitud fija de caracteres (ej: 'A   ' -> 4 bytes fijos)
NCHAR      -- Como CHAR pero soporta Unicode (ej: 'β' -> caracteres especiales)
VARCHAR2   -- Longitud variable, más eficiente (ej: 'Hola' -> solo usa 4 bytes)
NVARCHAR2  -- VARCHAR2 con soporte Unicode (ej: '你好' -> caracteres Unicode)
VARCHAR    -- Sinónimo de VARCHAR2 (obsoleto)
LONG       -- Texto largo hasta 2GB (ej: documentos grandes)
RAW        -- Datos binarios crudos (ej: imágenes)
LONG RAW   -- Datos binarios largos (ej: archivos grandes)
```

Los tipos de datos Numéricos:

```sql
NUMBER          -- Números con/sin decimales (ej: 123.45)
INT             -- Números enteros (ej: 1000)
DECIMAL         -- Similar a NUMBER (ej: 123.45)
BINARY_FLOAT    -- Punto flotante 32-bit (ej: 123.45f)
BINARY_DOUBLE   -- Punto flotante 64-bit (ej: 123.45d)
```

Los tipos de datos de fecha y hora:

```sql
DATE                            -- Fecha y hora (ej: '01-JAN-23')
TIMESTAMP                       -- Fecha y hora con precisión de nanosegundos (ej: '01-JAN-23 14:30:00.000')
TIMESTAMP WITH TIME ZONE        -- Timestamp con zona horaria (ej: '01-JAN-23 14:30:00 -05:00')
TIMESTAMP WITH LOCAL TIME ZONE  -- Timestamp convertido a zona local (ej: '01-JAN-23 14:30:00')
INTERVAL                        -- Período de tiempo (ej: INTERVAL '4' HOUR)
```

---

### 2. Crear Tabla de Datos

Estructura básica para crear una tabla:
```sql
CREATE TABLE nombre_tabla (
    nombre_columna1 tipo_dato [restricciones],
    nombre_columna2 tipo_dato [restricciones],
    ...
    [restricciones_tabla]
);
```

Ejemplos de tablas:
```sql
-- Tabla simple con código y descripción
CREATE TABLE TB_CATEGORIAS(
    CODIGO_CA NUMBER,          -- Identificador numérico de categoría
    DESCRIPCION VARCHAR2(30)   -- Nombre o descripción de la categoría
);

-- Tabla con diferentes precisiones numéricas
CREATE TABLE TB_MEDIDAS(
    CODIGO_ME NUMBER(3,0),      -- Código de 3 dígitos sin decimales (ej: 100)
    ABREVIATURA_ME VARCHAR2(3), -- Abreviatura de 3 caracteres (ej: 'KG')
    DESCRIPCION_ME VARCHAR2(20) -- Nombre de la medida (ej: 'Kilogramos')
);

-- Tabla con múltiples campos y tipos de datos
CREATE TABLE TB_ARTICULOS(
    CODIGO_AR NUMBER(5,0),       -- Código de 5 dígitos (ej: 10025)
    DESCRIPCION_AR VARCHAR2(50), -- Nombre del artículo
    MARCA_AR VARCHAR2(30),       -- Marca del producto
    CODIGO_ME NUMBER(3,0),       -- FK referencia a TB_MEDIDAS
    CODIGO_CA NUMBER(4,0),       -- FK referencia a TB_CATEGORIAS
    FECHA_ING DATE,              -- Fecha de ingreso
    STOCK_ACTUAL NUMBER(10,2)    -- Cantidad con 2 decimales (ej: 150.75)
);
```

---

### 3. Borrar Tablas de Datos

Estructura básica para borrar una tabla:

```sql
DROP TABLE nombre_tabla [CASCADE CONSTRAINTS | PURGE];
-- CASCADE CONSTRAINTS: Elimina todas las restricciones relacionadas
-- PURGE: Elimina la tabla permanentemente sin enviarla a la papelera
```
Ejemplos de borrado de tablas:
```sql
-- Borrado simple de tabla
DROP TABLE TB_MEDIDAS;

-- Borrado con eliminación de restricciones
DROP TABLE TB_CATEGORIAS CASCADE CONSTRAINTS;

-- Borrado permanente sin papelera
DROP TABLE TB_ARTICULOS PURGE;

-- Borrado de múltiples tablas en orden (por dependencias)
DROP TABLE TB_ARTICULOS;
DROP TABLE TB_MEDIDAS;
DROP TABLE TB_CATEGORIAS;
```

> **Nota:** Es importante borrar las tablas en el orden correcto para evitar errores por restricciones de integridad referencial.

---

### 4. Alteración de estructura de Tabla de Datos

#### Modificar columnas para agregar restricción NOT NULL
```sql
-- Agrega la restricción NOT NULL a columnas existentes
ALTER TABLE TB_CATEGORIAS MODIFY (CODIGO_CA NOT NULL);     -- Hace obligatorio el código de categoría
ALTER TABLE TB_CATEGORIAS MODIFY (DESCRIPCION_CA NOT NULL); -- Hace obligatoria la descripción
ALTER TABLE TB_MEDIDAS MODIFY (CODIGO_ME NOT NULL);        -- Hace obligatorio el código de medida
ALTER TABLE TB_ARTICULOS MODIFY (CODIGO_AR NOT NULL);      -- Hace obligatorio el código de artículo
```

#### Modificar tamaño de columna
```sql
-- Aumenta el tamaño de una columna VARCHAR2
ALTER TABLE TB_MEDIDAS MODIFY DESCRIPCION_ME VARCHAR2(30); -- Cambia tamaño de 20 a 30 caracteres
```

#### Eliminar columna
```sql
-- Elimina una columna existente de la tabla
ALTER TABLE TB_ARTICULOS DROP COLUMN MARCA_AR; -- Elimina la columna MARCA_AR y sus datos
```

#### Modificar tipo de dato y agregar valor por defecto
```sql
-- Modifica el tipo de dato y agrega valor por defecto
ALTER TABLE TB_ARTICULOS MODIFY CODIGO_AR NUMBER(6,0) DEFAULT 0;  -- Amplía dígitos y agrega default
ALTER TABLE TB_CATEGORIAS MODIFY CODIGO_CA NUMBER(4,0) DEFAULT 0; -- Agrega valor por defecto
ALTER TABLE TB_MEDIDAS MODIFY CODIGO_ME NUMBER(3,0) DEFAULT 0;    -- Agrega valor por defecto
```

#### Agregar nuevas columnas
```sql
-- Agrega una columna individual
ALTER TABLE TB_ARTICULOS ADD MARCA_AR VARCHAR2(30); -- Agrega columna para marca

-- Agrega múltiples columnas en una sola sentencia
ALTER TABLE TB_ARTICULOS ADD (
    CODIGO_ME NUMBER(3,0),  -- Agrega referencia a medidas
    CODIGO_CA NUMBER(4,0)   -- Agrega referencia a categorías
);
```

> **Nota:** Estas alteraciones son permanentes y pueden afectar la integridad de los datos existentes. Es recomendable hacer respaldo antes de ejecutarlas.

### 5. Agregar Clave Primaria a Tablas de Datos

La clave primaria es un identificador único para cada registro en una tabla. Se utiliza para garantizar que no haya duplicados y establecer relaciones entre tablas.

#### Agregar clave primaria simple
```sql
-- Agrega clave primaria a tabla de artículos
ALTER TABLE TB_ARTICULOS ADD PRIMARY KEY (CODIGO_AR);      -- Define CODIGO_AR como identificador único de artículos

-- Agrega clave primaria a tabla de categorías
ALTER TABLE TB_CATEGORIAS ADD PRIMARY KEY (CODIGO_CA);     -- Define CODIGO_CA como identificador único de categorías

-- Agrega clave primaria a tabla de medidas
ALTER TABLE TB_MEDIDAS ADD PRIMARY KEY (CODIGO_ME);        -- Define CODIGO_ME como identificador único de medidas
```

#### Agregar clave primaria con nombre personalizado
```sql
-- Agrega clave primaria con nombre específico
ALTER TABLE TB_ARTICULOS ADD CONSTRAINT PK_ARTICULOS 
    PRIMARY KEY (CODIGO_AR);    -- Crea PK nombrada para mejor mantenimiento

-- Agrega clave primaria compuesta (múltiples columnas)
ALTER TABLE TB_DETALLES ADD CONSTRAINT PK_DETALLES 
    PRIMARY KEY (CODIGO_AR, CODIGO_CA);    -- Crea PK usando combinación de columnas
```

> **Nota:** Una tabla solo puede tener una clave primaria y las columnas que la componen no pueden contener valores NULL.

### 6. Restricciones (Foreign Key)

Las llaves foráneas establecen relaciones entre tablas, garantizando la integridad referencial de los datos.

#### Agregar llave foránea simple
```sql
-- Agrega referencia a tabla de medidas
ALTER TABLE TB_ARTICULOS 
ADD CONSTRAINT FK_01                    -- Nombre de la restricción
FOREIGN KEY (CODIGO_ME)                -- Columna local que será FK
REFERENCES TB_MEDIDAS(CODIGO_ME);      -- Tabla y columna a la que hace referencia

-- Agrega referencia a tabla de categorías
ALTER TABLE TB_ARTICULOS 
ADD CONSTRAINT FK_02                    -- Nombre de la restricción
FOREIGN KEY (CODIGO_CA)                -- Columna local que será FK
REFERENCES TB_CATEGORIAS(CODIGO_CA);   -- Tabla y columna a la que hace referencia
```

#### Agregar llave foránea con opciones adicionales
```sql
-- Agrega FK con borrado en cascada
ALTER TABLE TB_ARTICULOS 
ADD CONSTRAINT FK_03
FOREIGN KEY (CODIGO_ME) 
REFERENCES TB_MEDIDAS(CODIGO_ME)
ON DELETE CASCADE;                      -- Borra registros relacionados automáticamente

-- Agrega FK que se establece como NULL al borrar referencia
ALTER TABLE TB_ARTICULOS 
ADD CONSTRAINT FK_04
FOREIGN KEY (CODIGO_CA) 
REFERENCES TB_CATEGORIAS(CODIGO_CA)
ON DELETE SET NULL;                     -- Establece NULL en registros relacionados
```

> **Nota:** Las llaves foráneas requieren que la columna referenciada sea clave primaria o tenga una restricción UNIQUE en la tabla padre.

### 7. Columnas IDENTITY Oracle

Las columnas IDENTITY permiten generar valores únicos automáticamente para cada nuevo registro, similar a AUTO_INCREMENT en otros sistemas.

#### Crear tabla con columna IDENTITY
```sql
-- Tabla con columna autoincremental simple
CREATE TABLE TB_ALMACENES (
    CODIGO_AL NUMBER(2,0) GENERATED ALWAYS AS IDENTITY,    -- Autoincrementa comenzando en 1
    DESCRIPCION_AL VARCHAR2(30)                           -- Descripción del almacén
);

-- Tabla con columna IDENTITY personalizada
CREATE TABLE TB_SUCURSALES (
    CODIGO_SU NUMBER GENERATED ALWAYS AS IDENTITY 
        (START WITH 100             -- Comienza en 100
         INCREMENT BY 10            -- Incrementa de 10 en 10
         MAXVALUE 999              -- Valor máximo permitido
         MINVALUE 100              -- Valor mínimo permitido
         CYCLE                     -- Vuelve a empezar al llegar al máximo
         CACHE 20),                -- Almacena 20 valores en memoria
    NOMBRE_SU VARCHAR2(50)         -- Nombre de la sucursal
);
```

#### Modificar una columna existente a IDENTITY
```sql
-- Convierte columna existente en IDENTITY
ALTER TABLE TB_ALMACENES MODIFY CODIGO_AL 
    GENERATED ALWAYS AS IDENTITY;    -- Convierte en autoincremental
```

> **Nota:** Las columnas IDENTITY son automáticamente NOT NULL y no se pueden modificar manualmente sus valores.

