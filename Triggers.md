# Index
🔷 [Triggers](#♣♦triggers♦♣) <br/>
🔷 [Procedimientos](#procedimientos) <br/>
🔷 [Indices](#indices) <br/>
🔷 [Vistas](#vistas) <br/>
🔷 [Ejemplo_Trigger](#trigger) <br/>

# 🔻Triggers🔻 
Los triggers son objetos que puedes crear en tu base de datos. Estos permiten **desencadenar *un evento* de forma automática** en la tabla a la cúal esten asociados. *En Java, por ejemplo, un evento es la acción que ocurre después de pulsar un botón.*

En nuestro caso, los triggers se pueden activar cuando: 
* Realizamos un ```INSERT``` en una tabla.
* Realizamos un ```DELETE``` en una tabla.
* Realizamos un ```UPDATE``` en una tabla.

### **Casos de uso**
Su principal uso es la realización de tareas de mantenimiento y administración de las BDD.
* **Copia de seguridad automática:** Puedes crear un trigger que se active después de cada inserción, actualización o eliminación de registros y realice una copia de la tabla en otro archivo o tabla. 
* **Notificaciones:** Puedes crear un trigger que capture el evento de inserción en la tabla de empleados (por ejemplo) y envíe un correo electrónico a 'X'.
* **Control de stock:** Puedes crear un trigger que se active cada vez que se actualiza la cantidad de un producto y verifique si ha alcanzado 'X' limite. Cuando se alcance ese limite, el trigger puede desencadenar cualquier evento, como solicitar más unidades del producto.

### **Sintaxis**
```SQL
    DELIMITER $$
    CREATE TRIGGER trigger_1 
    AFTER UPDATE  --> Cuando se ejecuta
    ON nombre_tabla FOR EACH ROW
    [BEGIN 
        *lo que hace trigger;* 
    END$$]
    [FOLLOWS] trigger_2

[BEFORE OR AFTER] --> Se desencadena antes o después de un INSERT, DELETE o UPDATE.
[INSERT, UPDATE, DELETE] --> Con que
[FOR EACH ROW]: Indica que el trigger se repite por cada registro.
[FOR EACH STATEMENT]: Indica que el trrigger se repite por cada sentencia SQL.
[FOLLOWS OR PRECEDES] 
```

1. En MySQL es imprescindible usar ```DELIMITER```.
2. Se declara el nombre del trigger.
3. Se establece cuando se va activar.
4. Se define la tabla en la que se va ejecutrar y columnas afectadas.
5. Se describe la acción del trigger que siempre comenzará con ```BEGIN``` y acabará con ```END```.
6. Si fuera necesario, se pueden utilizar las palabras ```FOLLOWS``` o ```PRECEDES``` para concatenar triggers. Hay que indicar el nombre del nuevo trigger para que funcione.
    * ```FOLLOWS```: Se ejecuta el trigger después del que has definido.
    * ```PRECEDES```: Se ejecuta el trigger antes del que has definido.

*Los delimitadores señalan el principio y el final de un trigger y puede ser cualquier signo que quieras establecer como: %%, ; $$...*

#### **SENTENCIAS ```OLD``` && ```NEW```**
Permiten diferenciar y hacer referencia a los datos antiguos (OLD) y a los nuevos (NEW) dentro del trigger. 

### **Ejemplo de uso con ```INSERT```**

  *Primero se crea la tabla auxiliar donde se van a volcar/guardar los datos*

```SQL
CREATE TABLE info_fabricante (
	nombre_f varchar (50),
    autor varchar(50),
    fecha date
);
```
*Despues se crea el trigger*

```SQL
DELIMITER $$
CREATE TRIGGER info_new_fabricantes_AI
AFTER INSERT
ON fabricante FOR EACH ROW
BEGIN
    INSERT INTO info_fabricante (nombre_f,autor,fecha) 
    VALUES (NEW.nombre,CURRENT_USER(),NOW());
END$$
```

```SQL
INSERT INTO `fabricante`(`nombre`) VALUES ('Apple');
```
![consulta0](https://i.gyazo.com/d8c66bc60c54950ed10ddee38a995739.png)

### **Ejemplo de uso con ```UPDATE```**

```SQL
CREATE TABLE info_producto (
	nombre_OLD varchar (50),
    precio_OLD double,
    nombre_NEW varchar (50),
    precio_NEW double,
    autor varchar(50),
    fecha date
);
```

```SQL
DELIMITER #
CREATE TRIGGER info_producto_BU
BEFORE UPDATE
ON producto FOR EACH ROW
BEGIN
	INSERT INTO info_producto (nombre_OLD,precio_OLD,nombre_NEW,precio_NEW,autor,fecha)
    VALUES (OLD.nombre,OLD.precio,NEW.nombre,NEW.precio,CURRENT_USER(),CURRENT_DATE());
END#
```

```SQL
UPDATE producto SET precio=500 WHERE id_producto = 9;
```
![consulta1](https://i.gyazo.com/e3d565e5bde7dc56533654a1e5c97789.png)

### **Instrucciones adicionales**
* ```SHOW TRIGGERS```: muestra los triggers de la BDD.
* ```DROP TRIGGER```nombre_trigger: elimina el trigger.
*La instrucción ```ALTER TRIGGER``` no existe, si se quiere modificar un trigger se debe borrar y crear de nuevo.*

---
## 🔵Procedimientos 
Un procedimiento consiste en crear un objeto que pueda almacenar instrucciones ```SQL```repetitivas. De esta forma, podemos crear una "consulta genérica" para las diferentes aplicaciones que usen la BDD ganando en eficiencia y seguridad. También puede servir para crear ```TRIGGERS``` más completos y complejos.

Un procedimiento se crea con la sentencia ```CREATE PROCEDURE``` y se invoca con ```CALL```. Además, al igual que los metodos de Java puede tener 0, 1 o varios parámetros. 

### **Parámetros de entrada, salida y entrada/salida**
En los procedimientos podemos tener tres tipos de palabras clave que definimos antes del parametro:
* Entrada ```IN```: Parámetros que no cambian su valor. Se considera paso por valor.
* Salida ```OUT```: Estos parámetros pueden cambiar su valor dentro del procedimiento. En programación sería equivalente al paso por referencia.
* Entrada/Salida: Combinación de los tipos ```IN``` y ```OUT```.

### Sintaxis
```SQL
DELIMITER #
CREATE PROCEDURE nombre_metodo
([IN,OUT,IN/OUT] definicion de variables)
BEGIN
    [CONSULTA]
END#
```
### **Ejemplo de procedimiento sencillo** ---> FALTA POR COMPLETAR

### **Ejemplo de uso con Triggers**
*Un trigger que contenga todo en una tabla con un ENUM['DELETE','INSERT','UPDATE']*

# 💠Indices💠
Un indice es una estructura de base de datos que optimiza las consultas por medio de un identificador único que se puede asignar a la fila de una tabla, permitiendo un rápido acceso a sus registros. Existen diferentes tipos de índices y diferentes formas de implementarlo en nuestra BDD.

## **Tipos de índices**

* Índices de clave primaria. Identifican de forma única una fila dentro de una tabla y no admiten valores nulos. Hacen uso de la palabra clave: ```PRIMARY KEY```
* Índices de clave ajena. Este índice hace referencia a una columna que es clave primaria en otra tabla. Hacen uso de la palabra clave: ```FOREIGN KEY```
* Índices únicos. Garantiza que los valores de una columna son únicos. Son similares a los índices de clave primaria, pero permiten valores nulos. Hacen uso de la palabra clave: ```UNIQUE```
* Entre otros...

### **Creación de índices al crear la tabla**

Se pueden definir los índices en el momento de la creación de la tabla:

```SQL
    CREATE TABLE PRODUCTOS (
        id_producto INTEGER AUTO_INCREMENT,
        nombre VARCHAR(50),
        precio DOUBLE NOT NULL,
        PRIMARY KEY (id_producto),
        UNIQUE (precio),
        INDEX idx_nombre (nombre)
    )

```

### **Creación de índices después de crear la tabla**


Es posible crear diferentes tipos de índices con la sentencia ```ALTER TABLE```:

```SQL
ALTER TABLE cliente ADD INDEX idx_nombre (nombre);
```

También se puede omitir el nombre:

```SQL
ALTER TABLE empleado ADD UNIQUE INDEX (email);
```

También es posible crear un ```ÍNDEX``` con su propio comando:

```SQL
CREATE UNIQUE INDEX precio_cli ON cliente (precio);
```

### **Instrucciones adicionales**
* ```SHOW INDEX```: muestra los índices de la BDD.
* ```DESC tabla```: muestra los indices de una tabla en particular.

*Ejemplo de ```DESCRIBE``` de la tabla Jardinería*

```SQL
DESCRIBE cliente;

+----------------------------+---------------+------+-----+---------+-------+
| Field                      | Type          | Null | Key | Default | Extra |
+----------------------------+---------------+------+-----+---------+-------+
| codigo_cliente             | int(11)       | NO   | PRI | NULL    |       |
| nombre_cliente             | varchar(50)   | NO   |     | NULL    |       |
| nombre_contacto            | varchar(30)   | YES  |     | NULL    |       |
| apellido_contacto          | varchar(30)   | YES  |     | NULL    |       |
| telefono                   | varchar(15)   | NO   |     | NULL    |       |
| fax                        | varchar(15)   | NO   |     | NULL    |       |
| linea_direccion1           | varchar(50)   | NO   |     | NULL    |       |
| linea_direccion2           | varchar(50)   | YES  |     | NULL    |       |
| ciudad                     | varchar(50)   | NO   |     | NULL    |       |
| region                     | varchar(50)   | YES  |     | NULL    |       |
| pais                       | varchar(50)   | YES  | MUL | NULL    |       |
| codigo_postal              | varchar(10)   | YES  |     | NULL    |       |
| codigo_empleado_rep_ventas | int(11)       | YES  | MUL | NULL    |       |
| limite_credito             | decimal(15,2) | YES  |     | NULL    |       |
+----------------------------+---------------+------+-----+---------+-------+
```

### **Ejemplo** --> FALTA POR COMPLETAR

Una vez creado el índice ejecutamos la consulta haciendo uso de ```MATCH``` y ```AGAINST```.
```SQL
SELECT *
FROM producto
WHERE MATCH(nombre, descripcion) AGAINST ('acero');
```

# 💠Vistas💠 ---> PROBAR QUE EL CODIGO FUNCIONA

Una vista es una **tabla virtual** generada a partir de la ejecución de varias consultas sobre una o más tablas. Una vista tiene la misma estructura de filas y columnas que cualquier otra tabla MySQL y se almacenan del mismo modo.

## Sintaxis

```SQL
CREATE VIEW nombre_vista
[LISTA DE COLUMNAS]
AS consulta
```

* nombre_vista: Representa el nombre de la tabla virtual.
* lista_columnas: Es el listado de columnas que creará y contendrá la vista.
* consulta: Se trata de una consulta ```SELECT``` que nos devolvuelven los datos que forman de la vista.

### **¿Cual es la ventaja de usar vistas?**

La mayor ventaja de utilizar vistas se obtiene en forma de **rendimiento**, ya que no estaremos generando constantemente una vista si ésta ya existe (al contrario de las tablas). Cuanto más complejas sean las consultas que se deben ejecutar para obtener la vista, mayor será la ganancia de rendimiento.  

Por lo tanto, una vista en términos generales, puede ayudarte a construir una interfaz simplificada y abstracta a una base de datos compleja.

### **Ejemplos**

*Crea una vista para que liste el nombre de los productos con un precio inferior a 100 euros*

```SQL
CREATE VIEW productos_baratos
AS SELECT precio 
FROM producto 
WHERE precio < 100;
```

*Crea una vista que muestre para cada uno de los pedidos, el código del pedido, la fecha, el nombre del cliente que realizó el pedido y el importe total del pedido.*

```SQL
CREATE OR REPLACE VIEW resumen_pedidos AS
SELECT 
    pedido.codigo_pedido,
    pedido.fecha_pedido,
    cliente.nombre_cliente,
    SUM(detalle_pedido.cantidad * detalle_pedido.precio_unidad) AS total

FROM cliente INNER JOIN pedido
    ON cliente.codigo_cliente = pedido.codigo_cliente
INNER JOIN detalle_pedido 
    ON pedido.codigo_pedido = detalle_pedido.codigo_pedido
GROUP BY pedido.codigo_pedido
```

**Cuando crees una vista**, podrás ejecutar consultas sobre ella como si de una tabla más se tratase. Por ejemplo:

```SQL
SELECT * FROM vistaProductosBarato;
```

La consulta anterior devolverá los siguientes resultados:

```SQL
+------------+
| nombre     |
+------------+
| Producto A |
| Producto B |
+------------+
```

Por otro lado, la consulta de Jardinería devolverá los siguientes resultados:

```SQL
+---------------+------------+----------------+-------+
| codigo_pedido |fecha_pedido| nombre_cliente | total |
+---------------+------------+----------------+-------+
| 123           |
| Producto B    |
+---------------+
```



# Ejemplo completo de ```TRIGGER``` <a name="trigger"></a>