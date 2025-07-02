/*### 1. Creación de tabla. (1,5 puntos)

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.

Necesitamos conocer la opinión de nuestros clientes sobre nuestros productos. Para ello se propone la creación de una nueva tabla llamada `Valoraciones`. Cada valoración versará sobre un producto y será realizada por un solo cliente. Cada producto podrá ser valorado por muchos clientes. Cada cliente podrá realizar muchas valoraciones. Un cliente no puede valorar más de una vez un mismo producto.

Para cada valoración necesitamos conocer la puntuación de 1 a 5 (sólo se permiten enteros) y la fecha en que se realiza la valoración.*/

CREATE TABLE valoraciones(
valoracionId INT AUTO_INCREMENT PRIMARY KEY,
productoId INT,
clienteId INT,
puntuacion INT CHECK(puntuacion >= 1 AND puntuacion <= 5),
fechaRealizacion DATE,
UNIQUE(clienteId, valoracionId),
FOREIGN KEY (productoId) REFERENCES productos(id)
	ON DELETE CASCADE
	ON UPDATE CASCADE,
FOREIGN KEY (clienteId) REFERENCES clientes(id)
	ON DELETE CASCADE
	ON UPDATE CASCADE
);

/*### 2. Consultas SQL (DQL). 3 puntos

Incluya su solución en el fichero `2.solucionConsultas.sql`.*/

/*#### 2.1. Devuelva el nombre del producto, el precio unitario y las unidades compradas para las 5 líneas de pedido con más unidades. (1 punto)*/

SELECT productos.nombre, lineaspedido.precio, lineaspedido.unidades FROM lineaspedido
JOIN productos ON lineaspedido.productoId = productos.id
ORDER BY lineaspedido.unidades DESC
LIMIT 5;

/*#### 2.3. Devuelva el nombre del empleado, la fecha de realización del pedido, el precio total del pedido y las unidades totales del pedido para todos los pedidos que de más 7 días de antigüedad desde que se realizaron. Si un pedido no tiene asignado empleado, también debe aparecer en el listado devuelto. (2 puntos)*/

SELECT usuarios.nombre, pedidos.fechaRealizacion, SUM(lineaspedido.precio) AS precioTotal, SUM(lineaspedido.unidades) AS unidadesTotales FROM lineaspedido
RIGHT JOIN pedidos ON lineaspedido.pedidoId = pedidos.id
RIGHT JOIN empleados ON pedidos.empleadoId = empleados.id
RIGHT JOIN usuarios ON usuarios.id = empleados.usuarioId
GROUP BY usuarios.nombre
HAVING TIMESTAMPDIFF(DAY, pedidos.fechaRealizacion, CURDATE())>7;

/*### 3. Procedimiento. Bonificar pedido retrasado. 3,5 puntos

Incluya su solución en el fichero `3.solucionProcedimiento.sql`.

Cree un procedimiento que permita bonificar un pedido que se ha retrasado debido a la mala gestión del empleado a cargo. Recibirá un identificador de pedido, asignará a otro empleado como gestor y reducirá un 20% el precio unitario de cada línea de pedido asociada a ese pedido. (1,5 puntos)

Asegure que el pedido estaba asociado a un empleado y en caso contrario lance excepción con el siguiente mensaje: (1 punto)

`El pedido no tiene gestor`.

Garantice que o bien se realizan todas las operaciones o bien no se realice ninguna. (1 punto)*/

DELIMITER //

CREATE OR REPLACE PROCEDURE ejercicio3C(pedidoId INT)
BEGIN 

DECLARE empleado INT;
DECLARE nuevoEmpleado INT;

-- manejo errores
DECLARE exit handler FOR SQLEXCEPTION 
BEGIN
	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'ERROR';
END ;

START TRANSACTION ;

SELECT empleados.id into empleado FROM empleados
JOIN pedidos ON pedidos.empleadoId = empleados.id
WHERE pedidoId = pedidos.id;

if empleado IS NULL then 
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El pedido no tiene gestor';
END if;

SELECT empleados.id into nuevoEmpleado FROM empleados
WHERE empleados.id != empleado
LIMIT 1;

UPDATE pedidos
SET pedidos.empleadoId = nuevoEmpleado
WHERE pedidos.id = pedidoId;

UPDATE lineaspedido
SET lineaspedido.precio = lineaspedido.precio*0.2
WHERE lineaspedido.pedidoId = pedidoId;

COMMIT ;

END //

DELIMITER ;


/*### 4. Trigger. 2 puntos

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `p_limitar_unidades_mensuales_de_productos_fisicos` que, a partir de este momento, impida la venta de más de 1000 unidades al mes de cualquier producto físico.*/

CREATE TRIGGER p_limitar_unidades_mensuales_de_productos_fisicos
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN
    
DECLARE res INT;

SELECT SUM(lineaspedido.unidades) FROM lineaspedido
JOIN pedidos ON pedidos.id = lineaspedido.pedidoId
WHERE pedidos.fechaRealizacion

ACABAR

END //

DELIMITER ;