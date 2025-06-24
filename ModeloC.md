
```sql
CREATE TABLE valoraciones(
id INT PRIMARY KEY AUTO_INCREMENT,
productoId INT NOT NULL,
clienteId INT NOT NULL,
puntuacion INT CHECK(puntuacion >= 1 AND puntuacion <= 5),
fechaRealizacion DATE,
UNIQUE(clienteId, productoId),
FOREIGN KEY (productoId) REFERENCES productos(id)
	ON DELETE CASCADE 
	ON UPDATE CASCADE,
FOREIGN KEY (clienteId) REFERENCES clientes(id)
	ON DELETE CASCADE 
	ON UPDATE CASCADE
);
```

### CONSULTA

```sql
SELECT productos.nombre, lineaspedido.precio, lineaspedido.unidades FROM lineaspedido
JOIN productos ON lineaspedido.productoId = productos.id
ORDER BY lineaspedido.unidades DESC
LIMIT 5;

SELECT usuarios.nombre, pedidos.fechaRealizacion, SUM(lineaspedido.precio * lineaspedido.unidades) AS precioTotal,SUM(lineaspedido.unidades) AS unidades FROM lineaspedido
JOIN pedidos ON pedidos.id = lineaspedido.pedidoId
RIGHT JOIN empleados ON pedidos.empleadoId = empleados.id
RIGHT JOIN usuarios ON empleados.usuarioId = usuarios.id
GROUP BY pedidos.id
HAVING TIMESTAMPDIFF(DAY, pedidos.fechaRealizacion, CURDATE())>7;
```
#DIFERENCIA DE HAVING Y WHERE

### PROCEDIMIENTO

```sql
DELIMITER //
CREATE OR REPLACE PROCEDURE bonificar(pedidoId INT)
BEGIN 

DECLARE empleado INT; 
DECLARE nuevoempleado INT;

-- manejo errores
DECLARE exit handler FOR SQLEXCEPTION 
BEGIN

	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error al bonificar pedido retrasado';
END ;

START TRANSACTION ;

SELECT empleadoId INTO empleado FROM pedidos
WHERE pedidos.id = pedidoId;

if empleado IS NULL then 
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error al bonificar pedido retrasado';
END if;

SELECT empleados.id INTO nuevoempleado FROM empleados
WHERE empleado != empleados.id
LIMIT 1;

UPDATE pedidos SET empleadoId = nuevoempleado
WHERE id = pedidoId;

UPDATE lineaspedido
SET precio = precio*0.8
WHERE pedidoId = pedidoId;

COMMIT ;

END //
DELIMITER ;
```