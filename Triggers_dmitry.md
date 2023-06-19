
1. [Triggers](#🔻1.Triggers🔻) 
    * [Casos de uso](#12-casos-de-uso)
    * [Sintaxis](#13-sintaxis)
    * [Sentencias OLD/NEW](#14-sentencias-old--new)
    * [Ejemplo 1](#ejemplo-de-trigger-con-insert)
    * [Ejemplo 2](#ejemplo-de-trigger-con-update)
    * [Instrucciones adicionales](#15-instrucciones-adicionales)
2. [Estructuras de Control](#2💠estructuras-de-control💠)
   * [Sintaxis](#21-sintaxis)
3. [Procedimientos](#3💠procedimientos💠)
   * [Parametros](#31-parámetros-de-entrada-salida-y-entradasalida)
   * [Sintaxis](#32-sintaxis)
   * [Ejemplo 1](#33-ejemplo-de-creación-de-una-variable-con-default)
   * [Ejemplo 2](#34-ejemplo-de-procedimiento)
4. [Índices](#4💠índices💠)
    * [Tipos](#41-tipos-de-índices)
    * [Modos de crear índices 1](#42-creación-de-índices-al-crear-la-tabla)
    * [Modos de crear índices 2](#43-creación-de-índices-después-de-crear-la-tabla)
    * [Instrucciones adicionales](#44-instrucciones-adicionales)
5. [Vistas](#5💠vistas💠)
    * [Sintaxis](#51-sintaxis)
    * [Ventajas de las vistas](#52-¿cual-es-la-ventaja-de-usar-vistas)
    * [Ejemplos](#53-ejemplos)
6. [Ejemplo Trigger](#6💠ejemplo-completo-de-trigger-💠)
    * [Creacion de nuevas columnas](#61-modificación-de-la-tabla)
    * [Creacion del trigger](#62-creamos-el-trigger)
    * [Pruebas](#63-consultas-de-prueba)



# 🔻1.Triggers🔻
Los triggers son objetos que puedes crear en tu base de datos. Estos permiten **desencadenar *un evento* de forma automática** en la tabla a la cúal están asociados. *En Java, por ejemplo, un evento es la acción que ocurre después de pulsar un botón.*

En nuestro caso, los triggers se pueden activar cuando: 
* Realizamos un ```INSERT``` en una tabla.
* Realizamos un ```DELETE``` en una tabla.
* Realizamos un ```UPDATE``` en una tabla.

### **1.2 Casos de uso**
Su principal uso es la realización de tareas de mantenimiento y administración de las BDD.
* **Copia de seguridad automática:** Puedes crear un trigger que se active después de cada inserción, actualización o eliminación de registros y realice una copia de la tabla en otro archivo o tabla. 
* **Notificaciones:** Puedes crear un trigger que capture el evento de inserción en la tabla de empleados (por ejemplo) y envíe un correo electrónico a 'X'.
* **Control de stock:** Puedes crear un trigger que se active cada vez que se actualiza la cantidad de un producto y verifique si ha alcanzado 'X' límite. Cuando se alcance ese límite, el trigger puede desencadenar cualquier evento, como solicitar más unidades del producto.

### **1.3 Sintaxis**
```SQL
    DELIMITER $$
    CREATE TRIGGER trigger_1 
    AFTER UPDATE  --> Cuando se ejecuta
    ON nombre_tabla FOR EACH ROW
    BEGIN 
        *lo que hace trigger;* 
    END$$

[BEFORE OR AFTER] --> Se desencadena antes o después de un INSERT, DELETE o UPDATE.
[INSERT, UPDATE, DELETE] --> Con que
[FOR EACH ROW]: Indica que el trigger se repite por cada registro.
[FOR EACH STATEMENT]: Indica que el trigger se repite por cada sentencia SQL.
```

1. En MySQL es imprescindible usar ```DELIMITER```.
2. Se declara el nombre del trigger.
3. Se establece cuando se va activar.
4. Se define la tabla en la que se va a ejecutar y las columnas afectadas.
5. Se describe la acción del trigger que siempre comenzará con ```BEGIN``` y acabará con ```END```.

*La palabra reservada ```DELIMITER``` señala el principio y el final de **un bloque de instrucciones** puede ser cualquier signo que quieras establecer como: %%, ; $$...*

###  **1.4 SENTENCIAS ```OLD``` && ```NEW```**
Permiten diferenciar y hacer referencia a los datos antiguos (OLD) y a los nuevos (NEW) dentro del trigger. 

###  **Ejemplo de trigger con ```INSERT```**

Ejemplo de ```TRIGGER``` para recopilar información de cada ```nuevo fabricante``` que se vaya a insertar en la tabla ```fabricante```. Se guardará también quien lo hizo en la columna de ```autor``` y cuando lo hizo en la columna ```fecha```.

  *Primero se crea la tabla auxiliar donde se van a volcar/guardar los datos*

```SQL
CREATE TABLE info_fabricante (
	nombre_f varchar (50),
    autor varchar(50),
    fecha date
);
```
*Después se crea el trigger*

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

###  **Ejemplo de trigger con ```UPDATE```**

En este ejemplo se creará una tabla secundaria para almacenar los posibles cambios sobre el ```nombre``` y el ```precio``` de la tabla ```producto```. 

Cada vez que alguien realice una sentencia ```UPDATE``` sobre la tabla productos se guardará el nombre y precio antiguos del producto, el nombre y precio nuevos, quién hizo los cambios y cuando.

*Creamos la tabla auxiliar*

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
*Creamos el trigger*
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
*Realizamos cambios para comprobar que funciona*
```SQL
UPDATE producto SET precio=500 WHERE id_producto = 9;
```
![consulta1](https://i.gyazo.com/e3d565e5bde7dc56533654a1e5c97789.png)

###  **1.5 Instrucciones adicionales**
* ```SHOW TRIGGERS```: muestra los triggers de la BDD.
* ```DROP TRIGGER```nombre_trigger: elimina el trigger.

*La instrucción ```ALTER TRIGGER``` no existe, si se quiere modificar un trigger se debe borrar y crear de nuevo.*

# 2.💠Estructuras de Control💠

En MySQL, existen varias estructuras de control que permiten realizar operaciones condicionales y repetitivas tanto en los triggers como en los procedimientos almacenados.

### 2.1 Sintaxis

{```Condicional IF```} ::  Permite ejecutar una acción o un bloque si se cumple una condición. Por ejemplo:

```SQL
IF condición THEN
    -- acciones a ejecutar si la condición es verdadera
ELSE
    -- acciones a ejecutar si la condición es falsa
END IF;
```

{```Bucle WHILE```} ::  Ejecuta una acción repetidamente mientras se cumple una condición específica. Por ejemplo:

```SQL
WHILE condición DO
    -- acciones a ejecutar mientras se cumpla la condición
END WHILE;
```

{```Bucle REPEAT```} ::  Es el do while de Java. El bloque de código se ejecuta al menos una vez. Por ejemplo:

```SQL
REPEAT
    -- acciones a ejecutar
UNTIL condición;
```

{```Bucle FOR```} :: Permite recorrer un rango de valores y ejecutar un bloque de código para cada valor. Por ejemplo:

```SQL
FOR variable IN rango DO
    -- acciones a ejecutar para cada valor
END FOR;
```

# 3.💠Procedimientos💠
Un procedimiento consiste en crear un objeto que pueda almacenar instrucciones ```SQL```repetitivas. *En cristiano, un método de Java para consultas SQL.*

Permite crear una "consulta genérica" para las diferentes aplicaciones que usen la BDD ganando en eficiencia y seguridad. También puede servir para crear ```TRIGGERS``` más completos y complejos.

Un procedimiento se crea con la sentencia ```CREATE PROCEDURE``` y se invoca con ```CALL```. Además, al igual que los métodos de Java puede tener 0, 1 o varios parámetros. 

### **3.1 Parámetros de entrada, salida y entrada/salida**
En los procedimientos podemos tener tres tipos de palabras clave que definimos antes del párametro:

{```IN```} :: Parámetros que no cambian su valor. Se considera paso por valor.

{```OUT```} :: Estos parámetros pueden cambiar su valor dentro del procedimiento. En programación sería equivalente al paso por referencia.

{```INOUT```} :: Combinación de los tipos IN y OUT que nos permite modificar la variable a nuestro antojo.

### **3.2 Sintaxis**
```SQL
DELIMITER #
CREATE PROCEDURE nombre_metodo(párametro?)
BEGIN
    *lo que hace el metodo;* 
END#
```

* Para asignar una nueva variable usaremos: ```DECLARE nombre_var```.
* Para asignar un valor a una variable usaremos podemos usar ```SET nombre_var``` o ```TIPO_DATO DEFAULT valor```.

###  3.3 Ejemplo de creación de una variable con ```DEFAULT```
```SQL
CREATE PROCEDURE ejemplo_txt()
    BEGIN
        DECLARE var INT DEFAULT 12345;
    END
```

### **3.4 Ejemplo de procedimiento**

En el siguiente ejemplo se crea un procedimiento que permite actualizar el precio de un producto determinado. Recibe por ```parametro``` el ```precio nuevo``` y la ```ID``` del producto que se quiere modificar. 

Además, se estableció una condición para que, en caso de que se introduzca un precio negativo, se haga una consulta interna y se re-asigne la variable al precio original del producto.

```SQL
DELIMITER $$
CREATE PROCEDURE actualizarPrecio (INOUT precioPrd double, id int)
BEGIN
	IF precioPrd < 0 THEN 
        	SET precioPrd = (SELECT precio FROM producto WHERE id_producto = id);
    END IF;
    
    UPDATE producto SET precio = precioPrd WHERE id_producto = id; --el return de SQL
END$$
```
*Llamamos al procedimiento...*
```SQL
CALL actualizarPrecio(nuevoPrecio,id);
```

PRECIO DEL ```IPHONE X``` ANTES DE LA MODIFICACIÓN:

![alt1](https://i.gyazo.com/768f943524b94f95c36ae320aed0c579.png)
<br/>

PRECIO DESPUÉS DE LA MODIFICACIÓN:
![image.png](https://i.gyazo.com/f5ec82ea2de7cbf32c3ebd5660d434bc.png)
![image.png](https://i.gyazo.com/15c48202bb469fb66dc06e1e60c5573b.png)

# 4.💠Índices💠
Un índice es una estructura de base de datos que permite realizar búsquedas o consultas con mayor rapidez. Existen diferentes tipos de índices y diferentes formas de implementarlos en nuestra BDD.

### **4.1 Tipos de índices**

* ```Índices de clave primaria.``` Identifican de forma única una fila dentro de una tabla y no admiten valores nulos. Hacen uso de la palabra clave: ```PRIMARY KEY```
* ```Índices únicos.``` Garantiza que los valores de una columna son únicos. Son similares a los índices de clave primaria, pero permiten valores nulos. Hacen uso de la palabra clave: ```UNIQUE```
* ```Índices compuestos.``` Se forman con varias columnas y permiten valores nulos.

### **4.2 Creación de índices al crear la tabla**

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

### **4.3 Creación de índices después de crear la tabla**


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

Por último, podemos crear un ```índex compuesto``` de varias columnas para evitar nombres duplicados. La clave evitará tener nombres | apellidos iguales y también permitirá dejar los campos vacios.

```SQL
CREATE UNIQUE INDEX myIndex ON empleado (nombre,apellido1,apellido2);
```

### **4.4 Instrucciones adicionales**
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

# 5.💠Vistas💠

Una vista es una **tabla virtual** generada a partir de la ejecución de varias consultas sobre una o más tablas. Una vista tiene la misma estructura de filas y columnas que cualquier otra tabla MySQL y se almacenan del mismo modo.

### 5.1 Sintaxis

```SQL
CREATE VIEW nombre_vista
[LISTA DE COLUMNAS]
AS consulta
```

* nombre_vista: Representa el nombre de la tabla virtual.
* lista_columnas: Es el listado de columnas que creará y contendrá la vista.
* consulta: Se trata de una consulta ```SELECT``` que nos devuelve los datos que forman de la vista.

### **5.2 ¿Cual es la ventaja de usar vistas?**

La mayor ventaja de utilizar vistas se obtiene en forma de **rendimiento**, ya que no estaremos generando constantemente una vista si ésta ya existe (al contrario de las tablas). Cuanto más complejas sean las consultas que se deben ejecutar para obtener la vista, mayor será la ganancia de rendimiento.  

Por lo tanto, una vista en términos generales, puede ayudarte a construir una interfaz simplificada y abstracta a una base de datos compleja.

### **5.3 Ejemplos**

*Crea una vista para que liste el nombre de los productos con un precio inferior a 100 euros*

```SQL
CREATE VIEW productos_baratos
AS SELECT precio 
FROM producto 
WHERE precio < 100;
```

*Crea una vista que muestre para cada uno de los pedidos, el código del pedido, la fecha, el nombre del cliente que realizó el pedido y el importe total del pedido.*

```SQL
CREATE VIEW resumen_pedidos AS
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
SELECT * FROM resumen_pedido;
```

```SQL
+---------------+------------+----------------+----------+
| codigo_pedido |fecha_pedido| nombre_cliente |  total   |
+---------------+------------+----------------+----------+
| 123           | 2023-12-20 | Alpaca         | 4534.56  |
| 124           | 2020-02-15 | Almerimar      | 67876.32 |
+---------------+------------+----------------+----------+
```



# 6.💠Ejemplo completo de ```TRIGGER``` 💠

Vamos a crear un ```TRIGGER``` que sea capaz de registrar los ascensos/descensos de los empleados. 

Para el ejemplo usaremos la base de datos de ```JARDINERIA``` y modificaremos la tabla empleado para que tenga 2 columnas adicionales: ```comentarios``` y ```fecha de modificación```.

### 6.1 Modificación de la tabla
*Añadimos la tabla de comentarios*
```SQL
ALTER TABLE empleado ADD COLUMN comentarios TEXT NULL;
```

*Añadimos la tabla de fecha_mod*
```SQL
ALTER TABLE empleado ADD COLUMN fecha_mod DATE NULL;
```
![modificacionesJardineria1](https://i.gyazo.com/85d59d1c0cdc16efe2d7562ee5984c84.png)

### 6.2 Creamos el trigger

```SQL
DELIMITER $$
CREATE TRIGGER ver_ascensos
BEFORE UPDATE ON empleado FOR EACH ROW
BEGIN
    IF OLD.PUESTO != NEW.puesto THEN
        IF NEW.puesto = 'Representante Ventas' THEN
                SET NEW.comentarios = 'El empleado ha sido asignado a ser Representante de ventas';
                SET NEW.fecha_mod = CURRENT_DATE();
            
        ELSEIF NEW.puesto = 'Secretaria' THEN
                SET NEW.comentarios = 'El empleado ha sido asignado a ser Secretaria';
                SET NEW.fecha_mod = CURRENT_DATE();
            
        ELSEIF NEW.puesto = 'Director Oficina' THEN
                SET NEW.comentarios = 'El empleado ha sido asignado a ser Director Oficina';
                SET NEW.fecha_mod = CURRENT_DATE();

        ELSEIF NEW.puesto = 'Subdirector Marketing' THEN
                SET NEW.comentarios = 'El empleado ha sido asignado a ser Subdirector Marketing';
                SET NEW.fecha_mod = CURRENT_DATE();

        ELSEIF NEW.puesto = 'Subdirector Ventas' THEN
                SET NEW.comentarios = 'El empleado ha sido asignado a ser Subdirector ventas';
                SET NEW.fecha_mod = CURRENT_DATE();
            
            ELSE
                SET NEW.puesto = OLD.puesto; 
                SET NEW.comentarios = 'Sin cambios';
                SET NEW.fecha_mod = CURRENT_DATE();
        END IF;
    END IF;    
END;$$
```

### 6.3 Consultas de prueba

*Cargo del empleado 4 antes de la modificación*

![altimg](https://i.gyazo.com/35e942346e1ec6b03b913b0b62f59d90.png)


*Cargo del **empleado 4** después de la modificación*

```SQL
UPDATE empleado SET puesto="Limpiadora" WHERE codigo_empleado = 4;
```

![alt23](https://i.gyazo.com/3be1b1c7be4580443bc99cc8e6b61c3f.png)

*Cargo del **empleado 3** después de la modificación*

```SQL
UPDATE empleado SET puesto="Director Oficina" WHERE codigo_empleado = 3;
```

![asd1](https://i.gyazo.com/7f291600d215f32dfee264a13e994d6f.png)