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

## Tipos de Funciones

### 1. Funciones de String o Cadena

Las funciones de cadena permiten manipular y transformar texto en Oracle.

#### Tabla DUAL
```sql
-- DUAL es una tabla especial de una sola fila útil para pruebas
SELECT * FROM dual;                    -- Muestra la única fila de DUAL
SELECT 'Hola Mundo' FROM dual;         -- Muestra texto literal
```

#### Funciones básicas de texto
```sql
-- Concatenación
SELECT CONCAT('Hola Josué', ' Como estas?') FROM dual;   -- Une dos cadenas

-- Conversión de caracteres
SELECT chr(64) FROM dual;              -- Convierte ASCII 64 a '@'

-- Transformación de mayúsculas/minúsculas
SELECT initcap('hola isra') FROM dual; -- Primera letra en mayúscula
SELECT upper('hola isra') FROM dual;   -- Todo a mayúsculas
SELECT lower('HOLA ISRA') FROM dual;   -- Todo a minúsculas
```

#### Funciones de relleno
```sql
-- Relleno izquierdo (LPAD)
SELECT LPAD('JOSUE',10,'x') FROM dual;         -- Rellena con 'x' a la izquierda
SELECT CONCAT('FACTURA-', 
       LPAD('1',5,'0')) FROM dual;             -- Útil para códigos con ceros

-- Relleno derecho (RPAD)
SELECT RPAD('JOSUE',10,'x') FROM dual;         -- Rellena con 'x' a la derecha
SELECT CONCAT(RPAD('P-',10,'0'),'1') FROM dual;-- Útil para formateo de texto
```

#### Funciones de limpieza
```sql
-- Eliminación de espacios
SELECT CONCAT(
    RTRIM('JOSUE         '),           -- Elimina espacios a la derecha
    LTRIM('         NABI')            -- Elimina espacios a la izquierda
) FROM dual;

-- Reemplazo de caracteres
SELECT REPLACE('LLL.YOUTUBE.COM',      -- Reemplaza caracteres específicos
    'L','W') FROM DUAL;                -- Cambia 'L' por 'W'
```

#### Funciones de extracción y medición
```sql
-- Extracción de subcadenas
SELECT SUBSTR('Curso de bases de datos',
    10, 14) FROM DUAL;                 -- Extrae porción del texto

-- Longitud de cadena
SELECT LENGTH('Curso de bases de datos') FROM DUAL;  -- Cuenta caracteres

-- Búsqueda de posición
SELECT INSTR('Curso de bases de datos', 
    'de') FROM DUAL;                   -- Encuentra posición de 'de'
```

#### Funciones de traducción
```sql
-- Traducción de caracteres
SELECT TRANSLATE('Curso de bases de datos',
    'oes','035') FROM DUAL;            -- Reemplaza múltiples caracteres
```

> [!NOTE]
> - DUAL es útil para probar funciones sin necesidad de tablas reales
> - Las funciones pueden anidarse para operaciones más complejas
> - La mayoría de funciones son sensibles a mayúsculas/minúsculas
> - Los índices en SUBSTR empiezan en 1, no en 0

---

### 2. Funciones Numéricas

Las funciones numéricas permiten realizar operaciones matemáticas y estadísticas con números.

#### Funciones de Redondeo
```sql
-- Redondeo a decimales específicos
SELECT ROUND(135.657,2) FROM DUAL;     -- Redondea a 135.66

-- Truncamiento a decimales específicos
SELECT TRUNC(135.657,2) FROM DUAL;     -- Trunca a 135.65
```

#### Funciones Matemáticas
```sql
-- Cálculo de módulo (residuo)
SELECT MOD(11,3) FROM DUAL;            -- Devuelve 2 (residuo de 11/3)
```

#### Funciones de Agregación
```sql
-- Contar registros
SELECT COUNT(CODIGO_AR) AS TOTAL_REGISTRO 
FROM tb_articulos;                     -- Cuenta total de artículos

-- Suma de valores
SELECT SUM(CODIGO_AR) 
FROM tb_articulos;                     -- Suma todos los códigos

-- Valor mínimo
SELECT MIN(CODIGO_AR) 
FROM tb_articulos;                     -- Obtiene el código más bajo

-- Valor máximo
SELECT MAX(CODIGO_AR) 
FROM tb_articulos;                     -- Obtiene el código más alto

-- Promedio
SELECT AVG(CODIGO_AR) 
FROM tb_articulos;                     -- Calcula el promedio de códigos
```

> [!NOTE] 
> - ROUND redondea al decimal más cercano
> - TRUNC simplemente corta los decimales
> - Las funciones de agregación trabajan sobre conjuntos de datos
> - COUNT ignora valores NULL por defecto
> - AVG calcula el promedio solo de valores no nulos

---

### 3. Funciones Fecha y Hora

Las funciones de fecha y hora permiten manipular y obtener información temporal en Oracle.

#### Funciones de fecha actual
```sql
-- Obtener fecha actual
SELECT CURRENT_DATE FROM DUAL;          -- Fecha actual
SELECT SYSDATE FROM DUAL;              -- Fecha y hora actual del sistema

-- Obtener timestamp actual
SELECT CURRENT_TIMESTAMP FROM DUAL;     -- Timestamp con zona horaria
SELECT SYSTIMESTAMP FROM DUAL;         -- Timestamp del sistema
```

#### Funciones de manipulación de meses
```sql
-- Agregar meses a una fecha
SELECT ADD_MONTHS(CURRENT_DATE, 1) FROM DUAL;      -- Suma 1 mes a fecha actual
SELECT ADD_MONTHS('25/04/2025', 2) FROM DUAL;      -- Suma 2 meses a fecha específica

-- Obtener último día del mes
SELECT LAST_DAY('25/04/2025') FROM DUAL;           -- Muestra 30/04/2025

-- Calcular diferencia en meses
SELECT MONTHS_BETWEEN('1/07/2025', 
    '1/01/2025') FROM DUAL;                        -- Muestra 6 meses
```

#### Funciones de manipulación de días
```sql
-- Obtener próximo día específico
SELECT NEXT_DAY('25/04/2025', 'MARTES') FROM DUAL; -- Próximo martes después del 25/04

-- Operaciones con días
SELECT to_date('19/12/2022')-2 FROM DUAL;          -- Resta 2 días a la fecha
```

#### Funciones de extracción
```sql
-- Extraer componentes de fecha
SELECT EXTRACT(YEAR FROM SYSDATE) FROM DUAL;        -- Obtiene el año
SELECT EXTRACT(MONTH FROM SYSDATE) FROM DUAL;       -- Obtiene el mes
SELECT EXTRACT(DAY FROM SYSDATE) FROM DUAL;         -- Obtiene el día
```

#### Funciones de formato
```sql
-- Formatear fecha como texto
SELECT CONCAT(to_char(SYSDATE), 
    ' Fecha del sistema') AS Fecha_actual FROM DUAL; -- Concatena con texto
```

> [!NOTE] 
> - SYSDATE incluye hora, CURRENT_DATE solo fecha
> - Los formatos de fecha pueden variar según la configuración regional
> - EXTRACT permite obtener partes específicas de una fecha
> - TO_CHAR convierte fechas a texto formateado

---

## Relaciones de datos entre tablas

### 1. INNER JOIN

El INNER JOIN combina registros de dos o más tablas cuando hay coincidencia en los campos relacionados.

#### Sintaxis básica
```sql
SELECT tabla1.campo1, tabla2.campo2
FROM tabla1
INNER JOIN tabla2 ON tabla1.campo = tabla2.campo;
```

#### Ejemplo completo con múltiples tablas
```sql
-- Consulta que relaciona artículos con sus categorías y medidas
SELECT 
    tb_articulos.codigo_ar,           -- Código del artículo
    tb_articulos.descripcion_ar,      -- Descripción del artículo
    tb_articulos.marca_ar,            -- Marca del artículo
    tb_medidas.descripcion_me,        -- Descripción de la medida
    tb_categorias.descripcion_ca,     -- Descripción de la categoría
    tb_articulos.stock_actual,        -- Stock disponible
    tb_articulos.fecha_registro       -- Fecha de registro
FROM tb_articulos
INNER JOIN tb_categorias             -- Unión con tabla categorías
    ON tb_articulos.codigo_ca = tb_categorias.codigo_ca
INNER JOIN tb_medidas               -- Unión con tabla medidas
    ON tb_articulos.codigo_me = tb_medidas.codigo_me;
```

> [!NOTE]  
> - INNER JOIN solo muestra registros donde hay coincidencia en ambas tablas
> - Se pueden encadenar múltiples INNER JOIN
> - Es recomendable usar alias para hacer el código más legible
> - El orden de las tablas en los JOIN puede afectar el rendimiento

---

### 2. LEFT JOIN

El LEFT JOIN retorna todos los registros de la tabla izquierda y los registros coincidentes de la tabla derecha. Si no hay coincidencia, los campos de la tabla derecha contendrán NULL.

#### Sintaxis básica
```sql
SELECT tabla_izq.campo1, tabla_der.campo2
FROM tabla_izq
LEFT JOIN tabla_der ON tabla_izq.campo = tabla_der.campo;
```

#### Ejemplo con categorías y artículos
```sql
-- Consulta que muestra todas las categorías y sus artículos (si existen)
SELECT 
    tb_categorias.codigo_ca,       -- Código de categoría
    tb_categorias.descripcion_ca,  -- Nombre de la categoría
    tb_articulos.descripcion_ar,   -- Descripción del artículo (puede ser NULL)
    tb_articulos.marca_ar         -- Marca del artículo (puede ser NULL)
FROM tb_categorias
LEFT JOIN tb_articulos           -- Mantiene todas las categorías
    ON tb_categorias.codigo_ca = tb_articulos.codigo_ca;
```

> [!NOTE]  
> - Muestra TODAS las categorías, tengan o no artículos asociados
> - Las categorías sin artículos mostrarán NULL en los campos de artículos
> - Útil para encontrar registros "huérfanos" o sin relaciones
> - El orden de las tablas es importante en LEFT JOIN

---

### 3. RIGHT JOIN

El RIGHT JOIN retorna todos los registros de la tabla derecha y los registros coincidentes de la tabla izquierda. Es el opuesto al LEFT JOIN.

#### Sintaxis básica
```sql
SELECT tabla1.campo1, tabla2.campo2
FROM tabla1
RIGHT JOIN tabla2 ON tabla1.campo = tabla2.campo;
```

#### Ejemplo con artículos y medidas usando alias
```sql
-- Consulta que muestra todas las medidas y sus artículos (si existen)
SELECT 
    A.codigo_ar,          -- Código del artículo (puede ser NULL)
    A.descripcion_ar,     -- Descripción del artículo (puede ser NULL)
    A.marca_ar,          -- Marca del artículo (puede ser NULL)
    B.descripcion_me     -- Descripción de la medida (siempre tiene valor)
FROM tb_articulos A      -- Alias A para tabla artículos
RIGHT JOIN tb_medidas B  -- Alias B para tabla medidas
    ON A.codigo_me = B.codigo_me;
```

> [!NOTE] 
> - Muestra TODAS las medidas, tengan o no artículos asociados
> - Los artículos sin medidas mostrarán NULL en sus campos
> - Los alias (A, B) hacen el código más corto y legible
> - RIGHT JOIN es menos común que LEFT JOIN pero cumple el mismo propósito invertido

---

## Procesos Orde By y Cláusula Where

### 1. SELECT ORDER BY

ORDER BY permite ordenar los resultados de una consulta según uno o más campos.

#### Ordenamiento básico descendente
```sql
-- Ordenar por código de forma descendente
SELECT *
FROM tb_articulos
ORDER BY codigo_ar DESC;    -- Mayor a menor
```

#### Ordenamiento ascendente (predeterminado)
```sql
-- Ordenar por marca alfabéticamente
SELECT *
FROM tb_articulos
ORDER BY marca_ar;          -- ASC es implícito si no se especifica
```

#### Ordenamiento por múltiples campos
```sql
-- Ordenar por descripción y marca
SELECT *
FROM tb_articulos
ORDER BY descripcion_ar DESC,  -- Primero por descripción descendente
         marca_ar;            -- Luego por marca ascendente
```

#### Ordenamiento por posición de columna
```sql
-- Ordenar usando número de columna
SELECT *
FROM tb_articulos
ORDER BY 5;                   -- Ordena por la quinta columna
```

> [!NOTE] 
> - ASC (ascendente) es el orden predeterminado
> - DESC (descendente) debe especificarse explícitamente
> - Se pueden combinar múltiples campos de ordenamiento
> - El orden de los campos afecta el resultado final
> - No es recomendable usar números de columna (menos mantenible)

---

### 2. CLÁUSULA WHERE

La cláusula WHERE permite filtrar registros basándose en condiciones específicas. Se puede usar en SELECT, UPDATE y DELETE.

#### Sintaxis básica
```sql
-- Con SELECT
SELECT * FROM tabla WHERE condición;

-- Con UPDATE
UPDATE tabla SET campo = valor WHERE condición;

-- Con DELETE
DELETE FROM tabla WHERE condición;
```

#### Ejemplos con SELECT
```sql
-- Filtrar por valor numérico
SELECT * 
FROM tb_articulos
WHERE stock_actual > 0;          -- Solo artículos con stock positivo

-- Filtrar con múltiples condiciones
SELECT * 
FROM tb_articulos
WHERE stock_actual > 0 
  AND codigo_ar = 5;            -- Stock positivo y código específico
```

#### Ejemplos con UPDATE
```sql
-- Actualizar registros filtrados
UPDATE tb_articulos 
SET stock_actual = stock_actual + 2    -- Incrementa stock en 2
WHERE marca_ar = 'ESTANDAR';           -- Solo para marca ESTANDAR
```

#### Ejemplos con DELETE
```sql
-- Borrar con condición OR
DELETE FROM tb_articulos 
WHERE marca_ar = 'ESTANDAR' 
   OR marca_ar = 'LENOVO';             -- Borra dos marcas específicas

-- Borrar usando IN (más eficiente)
DELETE FROM tb_articulos 
WHERE marca_ar IN ('ESTANDAR', 'LENOVO');  -- Forma alternativa más limpia
```

> [!NOTE]  
> - WHERE puede usar condiciones simples o compuestas
> - Se pueden combinar múltiples condiciones con AND y OR
> - Soporta operadores de comparación (>, <, =, !=, etc.)
> - IN es más eficiente que múltiples OR para listas de valores
> - Sin WHERE, la operación afecta a todos los registros

---

## Crear Vistas

Una vista es una tabla virtual basada en una consulta SQL. Permite simplificar consultas complejas y mejorar la seguridad.

### Sintaxis básica
```sql
CREATE VIEW nombre_vista AS
    subconsulta;
```

### Ejemplo completo de vista
```sql
-- Eliminar vista si existe
DROP VIEW VISTA_ARTICULOS;    

-- Crear vista de artículos con información relacionada
CREATE VIEW VISTA_ARTICULOS AS
SELECT a.codigo_ar,           -- Código del artículo
       a.descripcion_ar,      -- Descripción del artículo
       a.marca_ar,            -- Marca del artículo
       b.descripcion_me,      -- Descripción de la medida
       c.descripcion_ca,      -- Descripción de la categoría
       a.stock_actual,        -- Stock disponible
       a.fecha_registro,      -- Fecha de registro
       a.activo              -- Estado del artículo
FROM tb_articulos a          -- Alias 'a' para artículos
INNER JOIN tb_medidas b      -- Alias 'b' para medidas
    ON a.codigo_me = b.codigo_me
INNER JOIN tb_categorias c   -- Alias 'c' para categorías
    ON a.codigo_ca = c.codigo_ca
ORDER BY a.codigo_ar;        -- Ordenado por código

-- Consultar la vista
SELECT * FROM VISTA_ARTICULOS;
```

> [!NOTE]  
> - Las vistas no almacenan datos, solo la definición de la consulta
> - Se pueden usar como si fueran tablas normales
> - Útiles para simplificar consultas complejas
> - Ayudan a mantener la seguridad ocultando tablas base
> - Se pueden crear vistas sobre otras vistas

---

## Agrupamiento de Datos de Tablas

GROUP BY permite agrupar filas que tienen los mismos valores en las columnas especificadas y aplicar funciones de agregación a cada grupo.

### Casos de uso comunes:
1. Conteo de registros por grupo
2. Sumas totales por categoría
3. Promedios por grupo
4. Estadísticas agrupadas
5. Análisis de datos resumidos

### Ejemplos prácticos

#### Contar artículos por categoría
```sql
-- Cuenta cuántos artículos hay en cada categoría
SELECT 
    a.descripcion_ca,                    -- Nombre de la categoría
    COUNT(b.descripcion_ar) AS TOTAL_ARTICULOS   -- Cuenta artículos
FROM tb_categorias a
INNER JOIN tb_articulos b 
    ON a.codigo_ca = b.codigo_ca
GROUP BY a.descripcion_ca;              -- Agrupa por categoría
```

#### Contar artículos por marca
```sql
-- Cuenta cuántos artículos hay de cada marca
SELECT 
    marca_ar,                           -- Nombre de la marca
    COUNT(descripcion_ar) AS total_articulos  -- Cuenta artículos
FROM tb_articulos
GROUP BY marca_ar;                      -- Agrupa por marca
```

#### Sumar notas por curso
```sql
-- Suma todas las notas1 por curso
SELECT 
    curso,                              -- Nombre del curso
    SUM(NOTA1) AS total_suma_nota1     -- Suma de todas las nota1
FROM tb_notas_alumnos
GROUP BY curso;                         -- Agrupa por curso
```

> [!NOTE] 
> - GROUP BY debe incluir todas las columnas no agregadas en el SELECT
> - Se puede combinar con funciones de agregación (COUNT, SUM, AVG, etc.)
> - Útil para generar reportes resumidos
> - Se puede usar con HAVING para filtrar grupos
> - El orden de las columnas en GROUP BY no afecta el resultado

---

## Procedimientos Almacenados

### 1. Introducción
Un procedimiento almacenado es:
- Un conjunto de sentencias SQL
- Un conjunto de tareas o procesos
- Un conjunto de instrucciones codificadas
- Una unidad de código reutilizable

Se utilizan principalmente para operaciones:
- INSERT
- UPDATE
- DELETE

### 2. Sintaxis Básica
```sql
CREATE OR REPLACE PROCEDURE nombre_procedimiento (parametro IN tipo_datos)
    AS 
      variable_local tipo;
    BEGIN
      -- instrucciones
    END;
```

### 3. Ejemplo Básico
```sql
-- Procedimiento simple para insertar una medida
CREATE PROCEDURE INSERTAR_UN_REGISTRO_ME
AS
BEGIN
    INSERT INTO tb_medidas(codigo_me, abreviatura_me, descripcion_me)
    VALUES(3, 'LT', 'LITROS');
    COMMIT;
END;

-- Ejecutar el procedimiento
EXECUTE INSERTAR_UN_REGISTRO_ME;
```

### 4. Procedimiento con Múltiples Operaciones
```sql
-- Procedimiento que realiza INSERT y UPDATE
CREATE PROCEDURE CRUD_MEDIDAS_CATEGORIAS
AS
BEGIN
    -- Insertar nueva medida
    INSERT INTO tb_medidas(codigo_me, abreviatura_me, descripcion_me)
    VALUES(4, 'MT', 'METROS');
    
    -- Actualizar categoría existente
    UPDATE tb_categorias 
    SET descripcion_ca='EVENTOS 2025'
    WHERE codigo_ca=3;
    
    COMMIT;
END;
```

### 5. Tipos de Parámetros

1. **Parámetros IN**
   - Envían valores al procedimiento
   - No pueden ser modificados dentro del procedimiento
   - Son el tipo por defecto

2. **Parámetros OUT**
   - Devuelven valores del procedimiento
   - Similar al return en funciones
   - Se modifican dentro del procedimiento

3. **Parámetros IN OUT**
   - Combinación de IN y OUT
   - Pueden enviar y recibir valores
   - Se pueden modificar dentro del procedimiento

#### Sintaxis con Parámetros
```sql
CREATE OR REPLACE PROCEDURE nombre_procedimiento
(
    param1 [IN|OUT|IN OUT] tipo,
    param2 [IN|OUT|IN OUT] tipo
)
IS/AS
    -- Variables locales
BEGIN
    -- Cuerpo del procedimiento
EXCEPTION
    -- Manejo de excepciones
END nombre_procedimiento;
```

### 6. Ejemplos con Parámetros

#### Procedimiento con Parámetros IN
```sql
-- Actualizar categoría usando parámetros
CREATE OR REPLACE PROCEDURE ACTUALIZAR_CA(
    pCodigo IN NUMBER, 
    pDescripcion IN VARCHAR2)
AS
BEGIN
    UPDATE tb_categorias 
    SET descripcion_ca = pDescripcion
    WHERE codigo_ca = pCodigo;
    COMMIT;
END;

-- Ejemplos de uso
EXECUTE ACTUALIZAR_CA(3,'EVENTOS ESPECIALES');
EXECUTE ACTUALIZAR_CA(4,'EVENTOS GENERALES');
```

#### Procedimiento con Múltiples Parámetros
```sql
-- Insertar artículo con todos sus campos
CREATE OR REPLACE PROCEDURE GUARDAR_AR(
    pCodigo_ar NUMBER,
    pDescripcion_ar VARCHAR2,
    pMarca VARCHAR2,
    pCodigo_me NUMBER,
    pCodigo_ca NUMBER,
    pStock_Actual NUMBER,
    pFecha_Registro DATE,
    pActivo NUMBER)
AS
BEGIN
    INSERT INTO tb_articulos VALUES(
        pCodigo_ar, pDescripcion_ar, pMarca,
        pCodigo_me, pCodigo_ca, pStock_Actual,
        pFecha_Registro, pActivo
    );
    COMMIT;
END;
```

#### Procedimiento con Parámetro OUT
```sql
-- Buscar nota de alumno
CREATE OR REPLACE PROCEDURE BUSCAR_NOTA_AL(
    pCODIGO_AL IN NUMBER,
    pNOTA1 OUT NUMBER)
AS
BEGIN
    SELECT NOTA1 INTO pNOTA1
    FROM TB_NOTAS_ALUMNOS 
    WHERE CODIGO_AL = pCODIGO_AL;
END;

-- Uso con variables
VAR vNOTA1 NUMBER;
EXECUTE BUSCAR_NOTA_AL(2, :vNOTA1);
PRINT vNOTA1;
```

#### Procedimiento con Parámetro IN OUT
```sql
-- Buscar descripción de medida
CREATE PROCEDURE BUSCAR_DESCRIPCION_ME(
    p01 IN OUT VARCHAR2)
AS
BEGIN
    SELECT DESCRIPCION_ME INTO p01
    FROM TB_MEDIDAS
    WHERE ABREVIATURA_ME = p01;
END;
```

> [!NOTE]  
> - Los procedimientos pueden tener múltiples parámetros de diferentes tipos
> - COMMIT debe usarse para hacer permanentes los cambios
> - Las variables con ':' son variables de enlace (bind variables)
> - Es importante manejar excepciones en procedimientos productivos

---


