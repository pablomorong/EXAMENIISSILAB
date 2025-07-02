1. Creación de tabla. (1,5 puntos)
Incluya su solución en el fichero 1.solucionCreacionTabla.sql.

Queremos realizar un seguimiento del historial de cambios de precios de los productos. Para ello, cree una tabla llamada HistorialPrecios. Cada registro debe almacenar el identificador del producto, la fecha del cambio, el precio anterior y el precio nuevo.
Un producto puede tener múltiples cambios de precio a lo largo del tiempo.

⚠️ Restricciones:

El precio anterior y el nuevo deben ser positivos.

```sql
CREATE TABLE historialPrecios(
id INT AUTO_INCREMENT PRIMARY KEY,
productoId INT, 
fechaCambio DATE,
precioAnterior DECIMAL(10,2) CHECK(precioAnterior >= 0),
precioNuevo DECIMAL(10,2) CHECK(precioNuevo >= 0),
FOREIGN KEY (productoId) REFERENCES productos(id)
	ON UPDATE CASCADE
	ON DELETE CASCADE
);
```


2. Consultas SQL (DQL). 3 puntos
Incluya su solución en el fichero 2.solucionConsultas.sql.

2.1. Devuelva el nombre del producto, el precio medio al que ha sido vendido, y la cantidad total vendida. Solo para productos que han sido comprados más de 10 veces. (1 punto)

```sql
SELECT productos.nombre, AVG(lineaspedido.precio), SUM(lineaspedido.unidades) FROM lineaspedido
JOIN productos ON lineaspedido.productoId = productos.id
GROUP BY productos.nombre
HAVING sum(lineaspedido.unidades) > 10;
```


2.2. Devuelva el nombre del cliente, la cantidad total de productos que ha comprado, y el importe total que ha gastado en productos digitales. Los clientes que no hayan comprado productos digitales también deben aparecer. (2 puntos)
3. Procedimiento. Devolver pedido. (3,5 puntos)

```sql
SELECT usuarios.nombre, sum(lineaspedido.unidades), SUM(lineaspedido.precio * lineaspedido.unidades) FROM lineaspedido
RIGHT JOIN pedidos ON lineaspedido.pedidoId = pedidos.id
RIGHT JOIN clientes ON pedidos.clienteId = clientes.id
RIGHT JOIN usuarios ON usuarios.id = clientes.usuarioId
JOIN productos ON lineaspedido.productoId = productos.id
JOIN tiposproducto ON tiposproducto.id = productos.tipoProductoId
WHERE tiposproducto.nombre = 'Digitales'
GROUP BY usuarios.id;
```


Incluya su solución en el fichero 3.solucionProcedimiento.sql.

Cree un procedimiento que permita cancelar un pedido. El procedimiento recibirá un identificador de pedido, establecerá su fecha de envío a NULL, y pondrá la cantidad de todas sus líneas de pedido a 0. (1,5 puntos)

⚠️ Si el pedido ya fue enviado (es decir, su fecha de envío no es NULL), se debe lanzar una excepción con el mensaje:

No se puede cancelar un pedido ya enviado.

⚠️ Garantice que o bien se realizan todas las operaciones o ninguna. (2 puntos)


```sql
DELIMITER //

CREATE OR REPLACE PROCEDURE devolverPedido(pedidoId INT)

BEGIN 

DECLARE fechaEnvioPedido DATE;

DECLARE exit handler FOR SQLEXCEPTION 
BEGIN
	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'ERROR';
END ;

START TRANSACTION;

SELECT pedidos.fechaEnvio INTO fechaEnvioPedido FROM pedidos
WHERE pedidos.id = pedidoId;

if fechaEnvioPedido IS NOT NULL then
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se puede cancelar un pedido ya enviado.';
END if;

UPDATE pedidos
SET pedidos.fechaEnvio = NULL
WHERE pedidos.id = pedidoId;

UPDATE lineaspedido
SET lineaspedido.unidades = 0
WHERE lineaspedido.pedidoId = pedidoId;

COMMIT;

END //

DELIMITER ;
```

4. Trigger. 2 puntos
Incluya su solución en el fichero 4.solucionTrigger.sql.

Cree un trigger llamado p_prevenir_precios_irrealistas que evite que se inserten o actualicen productos con un precio menor a 0.01 o mayor a 10,000.

⚠️ Lanza una excepción con el mensaje:
Precio no válido: debe estar entre 0.01 y 10,000


```sql
    DELIMITER //

CREATE TRIGGER p_prevenir_precios_irrealistas
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN
    DECLARE precio DECIMAL(10,2);
    
    if NEW.precio < 0.01 OR NEW.precio > 10000 then
    	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Precio no válido: debe estar entre 0.01 y 10,000*/';
	END if;
   
END //

DELIMITER ;

DELIMITER //

CREATE TRIGGER p_prevenir_precios_irrealistas_update
BEFORE UPDATE ON LineasPedido
FOR EACH ROW
BEGIN
    DECLARE precio DECIMAL(10,2);
    
    if NEW.precio < 0.01 OR NEW.precio > 10000 then
    	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Precio no válido: debe estar entre 0.01 y 10,000*/';
	END if;
   
END //

DELIMITER ;
```