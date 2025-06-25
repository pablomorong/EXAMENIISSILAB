```sql
/*Necesitamos conocer los pagos que se realicen sobre los pedidos. Para ello se propone la creación de 
una nueva tabla llamada Pagos. Cada pedido podrá tener asociado varios pagos y cada pago solo 
corresponde con un pedido en concreto. 
Para cada pago necesitamos conocer la fecha de pago, la cantidad pagada (que no puede ser 
negativa) y si el pago ha sido revisado o no (por defecto no estará revisado).*/

CREATE TABLE pagos(
id INT PRIMARY KEY AUTO_INCREMENT,
pedidoId INT,
fechaPago DATE,
cantidadPagada INT CHECK(cantidadPagada >= 0),
esRevisado BOOLEAN,
UNIQUE(id, pedidoId),
FOREIGN KEY (pedidoId) REFERENCES pedidos(id)
	ON DELETE CASCADE 
	ON UPDATE CASCADE
);

/*2.1. Devuelva el nombre del del empleado, la fecha de realización del pedido y el nombre del cliente 
de todos los pedidos realizados este mes. (1 puntos) */

-- PONER DE LA TABLA USUARIOS UNA RELCIONADA CON CLIENTE SY OTRA CON EMPLEADOS

SELECT ue.nombre, pedidos.fechaRealizacion, uc.nombre FROM pedidos
JOIN empleados ON pedidos.empleadoId = empleados.id
JOIN usuarios ue ON empleados.usuarioId = ue.id
JOIN clientes ON clientes.usuarioId = pedidos.clienteId
JOIN usuarios uc ON uc.id = clientes.usuarioId;

/*2.2. Devuelva el nombre, las unidades totales pedidas y el importe total gastado de aquellos clientes 
que han realizado más de 5 pedidos en el último año. (2 puntos) */

SELECT usuarios.nombre, SUM(lineaspedido.unidades), SUM(lineaspedido.precio * lineaspedido.unidades) FROM lineaspedido
JOIN pedidos ON lineaspedido.pedidoId = pedidos.id
JOIN clientes ON clientes.id = pedidos.clienteId
JOIN usuarios ON usuarios.id = clientes.usuarioId
WHERE pedidos.fechaRealizacion >= CURDATE() - INTERVAL 1 YEAR 
GROUP BY usuarios.nombre
HAVING COUNT(DISTINCT pedidos.id) > 5; 

/*Cree un procedimiento que permita crear un nuevo producto con posibilidad de que sea para regalo. 
.
 Si el producto está destinado a regalo se creará un pedido con ese producto y costes 0€ para el 
cliente más antiguo. (1,5 puntos) 
Asegure que el precio del producto para regalo no debe superar los 50 euros y lance excepción si se 
da el caso con el siguiente mensaje: (1 punto) 
No se permite crear un producto para regalo de más de 50€. 
Garantice que o bien se realizan todas las operaciones o bien no se realice ninguna. (1 punto) */

DELIMITER //
CREATE OR REPLACE PROCEDURE ejercicio3(id INT, nombre VARCHAR(255), descripcion TEXT, precio DECIMAL , tipoProductoId INT, puedeVenderseMenores BOOLEAN, esRegalo BOOLEAN)
BEGIN

DECLARE productoNuevoId INT; 
DECLARE clienteAntiguoId INT;
DECLARE pedidoNuevoId INT;
 
-- manejo errores
DECLARE exit handler FOR SQLEXCEPTION 
BEGIN
	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'ERROR';
END ;

START TRANSACTION ;

if esRegalo = TRUE AND precio > 50 then 
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se permite crear un producto para regalo de más de 50€. ';
	
END if;

INSERT INTO productos(id, nombre, descripcion, precio, tipoProductoId, puedeVenderseAMenores)
VALUES(id, nombre, descripcion, precio, tipoProductoId, puedeVenderseMenores);

SET productoNuevoId = LAST_INSERT_ID();

if esRegalo = TRUE then 
SELECT clientes.id INTO clienteAntiguoId FROM clientes
ORDER BY clientes.fechaNacimiento ASC
LIMIT 1;

INSERT INTO pedidos(fechaRealizacion, fechaEnvio, direccionEntrega, comentarios, clienteId)
VALUES(CURDATE(), CURDATE(), 'Regalo', 'Regalo', 'Regalo', clienteAntiguoId);

SET pedidoNuevoId = LAST_INSERT_ID();

INSERT INTO lineaspedido(pedidoId,productoId, unidades, precio)
VALUES(pedidoNuevoId, productoNuevoId, 1, 0.00);

END if;

COMMIT;

END //

DELIMITER ;


/*Cree un trigger llamado t_limitar_importe_pedidos_de_menores que impida que, a partir de ahora, 
los pedidos realizados por menores superen los 500€. */



DELIMITER //

CREATE TRIGGER t_limitar_importe_pedidos_de_menores
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN
    DECLARE edadCliente INT;
    DECLARE totalPedido DECIMAL(10,2);

    -- Obtener la edad del cliente asociado al pedido
    SELECT TIMESTAMPDIFF(YEAR, c.fechaNacimiento, CURDATE())
    INTO edadCliente
    FROM Pedidos p
    JOIN Clientes c ON p.clienteId = c.id
    WHERE p.id = NEW.pedidoId;

    -- Obtener el importe total actual del pedido (sin la línea nueva)
    SELECT IFNULL(SUM(lp.unidades * lp.precio), 0)
    INTO totalPedido
    FROM LineasPedido lp
    WHERE lp.pedidoId = NEW.pedidoId;

    -- Añadir el importe de la nueva línea al total
    SET totalPedido = totalPedido + (NEW.unidades * NEW.precio);

    -- Si el cliente es menor y el total del pedido supera 500€, lanzar error
    IF edadCliente < 18 AND totalPedido > 500 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Los menores no pueden realizar pedidos de más de 500€';
    END IF;
END //

DELIMITER ;

```