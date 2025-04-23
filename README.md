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

> [!NOTE] 
> Es importante borrar las tablas en el orden correcto para evitar errores por restricciones de integridad referencial.

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

> [!NOTE] 
> Estas alteraciones son permanentes y pueden afectar la integridad de los datos existentes. Es recomendable hacer respaldo antes de ejecutarlas.

---

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

> [!NOTE] 
> Una tabla solo puede tener una clave primaria y las columnas que la componen no pueden contener valores NULL.

---

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

> [!NOTE] 
> Las llaves foráneas requieren que la columna referenciada sea clave primaria o tenga una restricción UNIQUE en la tabla padre.

---

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

> [!NOTE] 
> Las columnas IDENTITY son automáticamente NOT NULL y no se pueden modificar manualmente sus valores.

---

## Registro en Tablas

### 1. Insertar Registro en Tablas

La sintaxis básica para insertar registros es:
```sql
INSERT INTO nombre_tabla (columna1, columna2, ...) 
    VALUES (valor1, valor2, ...);
```

#### Insertar registros simples
```sql
-- Insertar categorías básicas
INSERT INTO tb_categorias(CODIGO_CA, DESCRIPCION_CA) 
    VALUES(1, 'OFICINAS');              -- Categoría para artículos de oficina

INSERT INTO tb_categorias(CODIGO_CA, DESCRIPCION_CA) 
    VALUES(2, 'HOGARES');               -- Categoría para artículos del hogar
    
INSERT INTO tb_categorias(CODIGO_CA, DESCRIPCION_CA) 
    VALUES(3, 'EVENTOS');               -- Categoría para artículos de eventos
```

#### Insertar registros con múltiples columnas
```sql
-- Insertar medidas con abreviatura
INSERT INTO tb_medidas(CODIGO_ME, ABREVIATURA_ME, DESCRIPCION_ME)
    VALUES(1,'UND', 'UNIDADES');        -- Medida para conteo individual
    
INSERT INTO tb_medidas(CODIGO_ME, ABREVIATURA_ME, DESCRIPCION_ME)
    VALUES(2,'MTS', 'METROS');          -- Medida para longitudes
    
INSERT INTO tb_medidas(CODIGO_ME, ABREVIATURA_ME, DESCRIPCION_ME)
    VALUES(3,'LTS', 'LITROS');          -- Medida para volúmenes
```

#### Insertar registros con claves foráneas
```sql
-- Insertar artículos relacionados a categorías y medidas
INSERT INTO tb_articulos(
    CODIGO_AR,                          -- ID del artículo
    DESCRIPCION_AR,                     -- Nombre del producto
    MARCA_AR,                           -- Marca del producto
    CODIGO_ME,                          -- Referencia a medidas
    CODIGO_CA                           -- Referencia a categorías
) VALUES(
    1,                                  -- Código único
    'COMPUTADOR',                       -- Descripción
    'LENOVO',                           -- Marca
    1,                                  -- Usa unidades
    1                                   -- Categoría oficinas
);

-- Insertar múltiples artículos para hogar
INSERT INTO tb_articulos(CODIGO_AR, DESCRIPCION_AR, MARCA_AR, CODIGO_ME, CODIGO_CA)
    VALUES(3, 'REFRIGERADOR', 'LG', 1, 2);      -- Electrodoméstico hogar

INSERT INTO tb_articulos(CODIGO_AR, DESCRIPCION_AR, MARCA_AR, CODIGO_ME, CODIGO_CA)
    VALUES(4, 'MICROONDAS', 'LG', 1, 2);        -- Electrodoméstico hogar
```

> [!NOTE] 
> Al insertar registros con claves foráneas, asegúrate de que los valores referenciados existan en las tablas padre.

---

### 2. Actualizar Registro en Tablas

La sintaxis básica para actualizar registros es:
```sql
UPDATE nombre_tabla 
SET campo = nuevo_valor 
[WHERE condición];
```

#### Actualizar registro específico
```sql
-- Actualizar marca de un artículo específico
UPDATE tb_articulos 
SET marca_ar = 'CANON'           -- Nuevo valor para la marca
WHERE codigo_ar = 2;             -- Solo afecta al artículo con código 2
```

#### Actualizar múltiples registros
```sql
-- Actualizar marca para varios artículos
UPDATE tb_articulos 
SET marca_ar = 'SAMSUNG'         -- Nueva marca para varios productos
WHERE codigo_ar IN (3,4);        -- Afecta a los artículos 3 y 4
```

#### Actualizar usando funciones
```sql
-- Actualizar descripción usando concatenación
UPDATE tb_articulos 
SET descripcion_ar = CONCAT('* ', descripcion_ar)  -- Agrega asterisco al inicio
WHERE marca_ar = 'ESTANDAR';                       -- Solo para productos estándar
```

> [!NOTE] 
> Si no se especifica una cláusula WHERE, la actualización afectará a todos los registros de la tabla. Siempre verifica la condición WHERE antes de ejecutar un UPDATE.

---

### 3. Borrar Registro en Tablas

La sintaxis básica para borrar registros es:
```sql
DELETE FROM nombre_tabla 
[WHERE condición];
```

#### Borrar registros específicos
```sql
-- Borrar registros por lista de IDs
DELETE FROM tb_articulos 
WHERE codigo_ar IN (5,6);        -- Borra artículos con códigos 5 y 6

-- Borrar registros por comparación
DELETE FROM tb_articulos 
WHERE codigo_ar > 4;             -- Borra artículos con código mayor a 4

-- Borrar registros por coincidencia de texto
DELETE FROM tb_articulos 
WHERE marca_ar = 'ESTANDAR';     -- Borra todos los artículos de marca estándar
```

#### Manejo de restricciones al borrar
```sql
-- Intento de borrar registro referenciado (generará error)
DELETE FROM tb_medidas 
WHERE codigo_me = 1;             -- Error: viola integridad referencial

-- Borrado exitoso de registros no referenciados
DELETE FROM tb_medidas 
WHERE codigo_me IN (2,3,4);      -- Éxito: registros sin referencias
```

> [!NOTE] 
> - Si no se especifica WHERE, se borrarán todos los registros de la tabla
> - No se pueden borrar registros que están siendo referenciados por otras tablas (FK)
> - El borrado de registros es permanente y no se puede deshacer

---

### 4. Consulta de Registro en Tablas

La sintaxis básica para consultar registros es:
```sql
SELECT [columnas | *] 
FROM nombre_tabla 
[WHERE condición];
```

#### Consultas básicas
```sql
-- Consultar todos los campos de una tabla
SELECT * FROM tb_medidas;              -- Muestra todas las medidas

-- Consultar todos los campos de otra tabla
SELECT * FROM tb_articulos;            -- Muestra todos los artículos
```

#### Consultas con filtros
```sql
-- Filtrar por valor específico
SELECT * 
FROM tb_articulos 
WHERE codigo_ca = 2;                   -- Solo artículos de categoría 2

-- Seleccionar columnas específicas con filtro
SELECT codigo_ar,                      -- Solo muestra estas tres columnas
       descripcion_ar,
       marca_ar
FROM tb_articulos 
WHERE codigo_ca = 2;                   -- Solo artículos de categoría 2
```

#### Consultas con operadores lógicos
```sql
-- Filtrar excluyendo valores
SELECT *
FROM tb_articulos 
WHERE NOT marca_ar IN ('LENOVO', 'CANON');   -- Excluye estas marcas

-- Búsqueda por patrón
SELECT *
FROM tb_articulos 
WHERE descripcion_ar LIKE 'R%';              -- Productos que empiezan con 'R'
```

> [!NOTE]
> - El asterisco (*) selecciona todas las columnas
> - LIKE permite búsquedas con comodines (% cualquier cantidad de caracteres)
> - IN permite especificar múltiples valores posibles

---

## Operadores en oracle

### 1. Operadores Aritméticos 

Los operadores aritmeticos permiten realizar cálculos con valores numéricos.

Son:

- multiplicación `(*)`
- división       `(/)`
- suma           `(+)`
- resta          `(-)`

Primero creamos una tabla de ejemplo para demostrar los operadores:
```sql
-- Crear tabla de notas
CREATE TABLE tb_notas_alumnos(
    CODIGO_AL NUMBER(3,0),           -- ID del alumno
    NOMBRE_AL VARCHAR2(50),          -- Nombre del alumno
    CURSO VARCHAR2(30),              -- Nombre del curso
    NOTA1 NUMBER(2,0),              -- Primera nota
    NOTA2 NUMBER(2,0),              -- Segunda nota
    NOTA3 NUMBER(2,0),              -- Tercera nota
    PROMEDIO NUMBER(5,2)            -- Promedio final
);

-- Insertar datos de prueba
INSERT INTO tb_notas_alumnos VALUES(1,'JOSUE','IPC2',13,14,15,0);
INSERT INTO tb_notas_alumnos VALUES(2,'JOSEPH','IPC2',10,12,7,0);
INSERT INTO tb_notas_alumnos VALUES(3,'ANDREY','IPC2',15,15,15,0);
```

#### Ejemplos con Suma (+)
```sql
-- Suma simple de notas
SELECT codigo_al, nombre_al, NOTA1+NOTA2+NOTA3 
FROM tb_notas_alumnos;

-- Suma con alias de columna
SELECT codigo_al, nombre_al, 
       (NOTA1+NOTA2+NOTA3) AS SUMATORIA_NOTAS 
FROM tb_notas_alumnos;
```

#### Ejemplos con Resta (-)
```sql
-- Resta simple
SELECT codigo_al, nombre_al, 
       (NOTA1+NOTA2)-NOTA3 
FROM tb_notas_alumnos;

-- Resta con múltiples alias
SELECT codigo_al,
       nombre_al,
       (NOTA1+NOTA2) AS ACOMULADO_NOTA1_NOTA2,
       NOTA3,
       (NOTA1+NOTA2)-NOTA3 AS ACOMULADO_MENOS_NOTA3
FROM tb_notas_alumnos;
```

#### Ejemplos con Multiplicación (*)
```sql
-- Multiplicar dos notas
SELECT codigo_al, 
       nombre_al,
       nota1*nota2 AS PRODUCTO_NOTAS
FROM tb_notas_alumnos;
```

#### Ejemplos con División (/)
```sql
-- División simple
SELECT codigo_al, 
       nombre_al,
       nota1/nota2 AS DIVISION_NOTAS
FROM tb_notas_alumnos;

-- Cálculo de promedio simple
SELECT codigo_al,
       nombre_al,
       curso,
       (nota1+nota2+nota3)/3 AS PROMEDIO
FROM tb_notas_alumnos;

-- Promedio redondeado
SELECT codigo_al,
       nombre_al,
       curso,
       ROUND(((nota1+nota2+nota3)/3),2) AS PROMEDIO
FROM tb_notas_alumnos;
```

> [!NOTE] 
> - ROUND() redondea al número de decimales especificado
> - AS permite renombrar las columnas resultantes
> - Los operadores pueden combinarse en una misma expresión

---

### 2. Operadores Relacionales o de Comparación

Los operadores relacionales permiten comparar valores y devolver resultados booleanos (verdadero o falso).

Los operadores son:

- igual a `=`
- diferente de `<>` o `!=`
- mayor que `>`
- menor que `<`
- mayor o igual que `>=`
- menor o igual que `<=`

Utilizaremos la tabla de notas creada anteriormente para mostrar ejemplos de cada operador relacional.

#### Igual a (=)
```sql
-- Buscar notas exactas
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 = 15;    -- Muestra alumnos con nota1 igual a 15
```

#### Diferente de (<> o !=)
```sql
-- Buscar notas diferentes
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 <> 15;   -- Muestra alumnos con nota1 diferente de 15
-- También se puede usar !=
-- WHERE nota1 != 15;
```

#### Mayor que (>)
```sql
-- Buscar notas superiores
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 > 12;    -- Muestra alumnos con nota1 mayor a 12
```

#### Menor que (<)
```sql
-- Buscar notas inferiores
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 < 15;    -- Muestra alumnos con nota1 menor a 15
```

#### Mayor o igual que (>=)
```sql
-- Buscar notas mayores o iguales
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 >= 13;   -- Muestra alumnos con nota1 mayor o igual a 13
```

#### Menor o igual que (<=)
```sql
-- Buscar notas menores o iguales
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 <= 13;   -- Muestra alumnos con nota1 menor o igual a 13
```

> [!NOTE] 
> - Los operadores relacionales se usan principalmente en la cláusula WHERE
> - Pueden combinarse con operadores lógicos (AND, OR)
> - Son útiles para filtrar datos según condiciones específicas

---

### 3. Operadores Concatenación


Para concatenar cadenas de texto en Oracle, se utiliza el operador `||` (doble barra vertical).

#### Concatenación simple
```sql
-- Unir nombre del alumno con el curso
SELECT codigo_al,
       nombre_al || ' - ' || curso        -- Concatena con guión entre textos
FROM tb_notas_alumnos;
```

#### Concatenación con alias
```sql
-- Unir textos y asignar nombre a la columna resultante
SELECT codigo_al,
       (nombre_al || ' - ' || curso) AS NOMBRE_CURSO    -- Nombra la columna concatenada
FROM tb_notas_alumnos;
```

> [!NOTE] 
> - El operador `||` puede unir dos o más cadenas
> - Se pueden incluir espacios y caracteres especiales entre comillas simples
> - El resultado puede asignarse a un alias usando AS
> - Los paréntesis ayudan a agrupar las concatenaciones

---

### 4. Operadores Lógicos

Los operadores lógicos permiten combinar condiciones en consultas SQL. Los más comunes son:

- AND: ambas condiciones deben ser verdaderas
- OR: al menos una condición debe ser verdadera
- NOT: invierte el resultado de una condición (verdadero a falso y viceversa)
- (): agrupa condiciones para evaluar primero lo que está dentro de los paréntesis

Tabla de verdad operador AND:

| OP1 | OP2 | SALIDA |
| --- | --- | ------ |
| V   | V   | V      |
| V   | F   | F      |
| F   | V   | F      |
| F   | F   | F      |

Tabla de verdad operador OR:

| OP1 | OP2 | SALIDA |
| --- | --- | ------ |
| V   | V   | V      |
| V   | F   | V      |
| F   | V   | V      |
| F   | F   | F      |

Tabla de verdad operador NOT:

| OP1 | SALIDA |
| --- | ------ |
| V   | F      |
| F   | V      |

#### Operador AND
```sql
-- Buscar notas que cumplan dos condiciones
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 > 13 AND nota2 > 13;    -- Ambas notas deben ser mayores a 13

-- Otro ejemplo con AND
SELECT * 
FROM tb_notas_alumnos 
WHERE nota1 >= 13 AND nota2 >= 13;  -- Ambas notas deben ser 13 o más
```

#### Operador OR
```sql
-- Buscar notas que cumplan al menos una condición
SELECT * 
FROM tb_notas_alumnos 
WHERE nota2 = 14 OR nota3 > 10;     -- Nota2 es 14 o nota3 mayor a 10
```

#### Operador NOT
```sql
-- Invertir una condición
SELECT * 
FROM tb_notas_alumnos 
WHERE NOT nota1 >= 15;              -- Notas menores a 15 (invierte >= 15)
```

#### Combinación de operadores con paréntesis
```sql
-- Combinar múltiples condiciones
SELECT * 
FROM tb_notas_alumnos 
WHERE (nota1 >= 9 AND nota1 <= 13)  -- Nota1 entre 9 y 13
   OR nota2 > 12;                   -- O nota2 mayor a 12
```

> [!NOTE]  
> - Los paréntesis determinan el orden de evaluación
> - Se pueden combinar múltiples operadores lógicos
> - AND tiene precedencia sobre OR
> - NOT afecta a la condición que le sigue

---

## Commit y Rollback

### COMMIT
El comando COMMIT guarda permanentemente todos los cambios realizados en la transacción actual.

```sql
-- Insertar un nuevo registro
INSERT INTO tb_medidas VALUES(2,'KGS', 'KILOGRAMOS');

-- Guardar los cambios permanentemente
COMMIT;  -- Los cambios ya no se pueden deshacer
```

### ROLLBACK
El comando ROLLBACK deshace todos los cambios no confirmados desde el último COMMIT.

```sql
-- Insertar un registro (cambio temporal)
INSERT INTO tb_medidas VALUES(3,'LBS', 'LIBRAS');

-- Si nos arrepentimos, podemos deshacer
ROLLBACK;  -- El registro no se guardará
```

> [!NOTE]  
> - COMMIT hace permanentes los cambios
> - ROLLBACK deshace cambios no confirmados
> - Los cambios después de un COMMIT no se pueden deshacer con ROLLBACK
> - Una sesión nueva siempre puede ver los cambios confirmados con COMMIT

---

