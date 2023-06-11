# ♣♦Triggers♦♣ 
Los triggers son objetos que puedes crear en tu base de datos. Estos permiten desencadenar *un evento* de forma automática en la tabla a la cúal esten asociados. *En Java, por ejemplo, un evento es la acción que ocurre después de pulsar un botón.*

En nuestro caso, los triggers se pueden activar cuando: 
* Realizamos un ```INSERT``` en una tabla.
* Realizamos un ```DELETE``` en una tabla.
* Realizamos un ```UPDATE``` en una tabla.

### Casos de uso
Su principal uso es la realización de tareas de mantenimiento y administración de las BDD.
* **Copia de seguridad automática:** Puedes crear un trigger que se active después de cada inserción, actualización o eliminación de registros y realice una copia de la tabla en otro archivo o tabla. 
* **Notificaciones:** Puedes crear un trigger que capture el evento de inserción en la tabla de empleados (por ejemplo) y envíe un correo electrónico a 'X'.
* **Control de stock:** Puedes crear un trigger que se active cada vez que se actualiza la cantidad de un producto y verifique si ha alcanzado 'X' limite. Cuando se alcance ese limite, el trigger puede desencadenar cualquier evento, como solicitar más unidades del producto.

### Sintaxis
```SQL
    DELIMITER #
    CREATE TRIGGER nombre
    trigger_time + trigger_event --> Cuando se ejecuta
    AFTER UPDATE --> ejemplo
    ON tbl_name FOR EACH ROW
    trigger_body [BEGIN *evento* END]

trigger_time:  [BEFORE OR AFTER]
trigger_event: [INSERT OR UPDATE OR DELETE]
trigger_order: [FOLLOWS OR PRECEDES] 
```
1. Primero se establece un delimitador (Como en un buffer, y puede ser cualquier signo que quieras establecer como: %%, $$, ; etc..).
2. Se declara el nombre del trigger.
3. Se establece cuando se va activar.
4. Se define la tabla en la que se va ejecutrar y columnas afectadas.
5. Se describe la acción del trigger que siempera comenzará con ```BEGIN``` y acabará con ```END```.