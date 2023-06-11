# ♣♦Triggers♦♣ 
Los triggers son objetos que puedes crear en tu base de datos. Estos permiten desencadenar *un evento* de forma automática en la tabla a la cúal esten asociados. *En Java, por ejemplo, un evento es la acción que ocurre después de pulsar un botón.*

En nuestro caso, los triggers se pueden activar cuando: 
* Realizamos ```INSERT``` en una tabla
* Realizamos ```DELETE``` en una tabla
* Realizamos ```UPDATE``` en una tabla

###Casos de uso
* **Copia de seguridad automática:** Puedes crear un trigger que se active después de cada inserción, actualización o eliminación de registros y realice una copia de la tabla en otro archivo. 
* **Notificaciones:** Puedes crear un trigger que capture el evento de inserción en la tabla de empleados y envíe un correo electrónico a 'X'.
* **Control de valores:** Puedes crear un trigger que se active cada vez que se actualiza la cantidad de un producto y verifique si ha alcanzado 'X' limite. Cuando se alcance ese limite, el trigger puede desencadenar cualquier evento, como solicitar más unidades del producto.