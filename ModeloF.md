1. Creación de tabla. (1,5 puntos)
 Incluya su solución en el fichero 1.solucionCreacionTabla.sql.
 Se desea registrar incidencias ocurridas durante el procesamiento de los pedidos. Para ello, cree
 una tabla llamada Incidencias. 
Cada pedido puede tener varias incidencias, pero cada incidencia pertenece a un único pedido.
 Cada incidencia tendrá:- Una descripción (texto no vacío)- Una fecha de registro- Un campo que indique si la incidencia fue resuelta (valor por defecto: no resuelta)

```sql
CREATE TABLE incidencias(
incidenciaId INT PRIMARY KEY AUTO_INCREMENT,
pedidoId INT,
descripcion TEXT NOT NULL,
fechaRegistro DATE,
estaResuelta BOOLEAN DEFAULT FALSE,
UNIQUE(pedidoId),
FOREIGN KEY (pedidoId) REFERENCES pedidos(id)
	ON DELETE CASCADE
	ON UPDATE CASCADE
);
```
2. Consultas SQL (DQL). (3 puntos)
 Incluya su solución en el fichero 2.solucionConsultas.sql.
 
 2.1. Devuelva el nombre del cliente, el total gastado en pedidos que fueron enviados, y el número
 total de productos comprados por dicho cliente. 
     Solo se incluirán clientes que hayan gastado más de 300?. (1 punto)
```sql     
   SELECT usuarios.nombre, SUM(lineaspedido.precio), SUM(lineaspedido.unidades) FROM lineaspedido
   JOIN pedidos ON pedidos.id = lineaspedido.pedidoId
   JOIN clientes ON clientes.id = pedidos.clienteId
   JOIN usuarios ON usuarios.id = clientes.usuarioId
	WHERE pedidos.fechaEnvio IS NOT NULL
   GROUP BY usuarios.id
   HAVING SUM(lineaspedido.precio) > 300;
   ```  
2.2. Devuelva el nombre del producto, el número de pedidos en los que ha sido incluido y el total
 facturado por ese producto en el último trimestre. (2 puntos)
 ```sql
 SELECT productos.nombre, COUNT(DISTINCT lineaspedido.pedidoId), (lineaspedido.unidades * lineaspedido.precio) FROM lineaspedido
 JOIN productos ON lineaspedido.productoId = productos.id
 JOIN pedidos ON pedidos.id = lineaspedido.pedidoId
 WHERE pedidos.fechaRealizacion >= CURDATE() - INTERVAL 3 MONTH
 GROUP BY productos.id;
 ```
## DISTINCT -> Porque queremos saber en cuántos pedidos diferentes ha aparecido cada producto.
 
 
3. Procedimiento. Generar promoción de producto. (3,5 puntos)
 Incluya su solución en el fichero 3.solucionProcedimiento.sql.
 Cree un procedimiento que permita aplicar una promoción a un producto, reduciendo su precio un
 15% y enviando automáticamente un pedido gratuito 
(precio 0?) al cliente más activo (el que haya hecho más pedidos en el último año). (1,5 puntos)
 Antes de aplicar la promoción, verifique que el nuevo precio no será inferior a 5?. Si lo es, debe
 lanzarse una excepción con el siguiente mensaje:
    El precio promocionado no puede ser inferior a 5?. (1 punto)
 Asegure que todas las operaciones del procedimiento se ejecutan de forma atómica (transacción
 completa o ninguna). (1 punto)
 ```sql
 DELIMITER //
 
 CREATE OR REPLACE PROCEDURE ejercicio3(productoId INT, fechaEnvio DATE, direccionEntrega VARCHAR(255), empleadoId INT)
 BEGIN 
 
 DECLARE precioAntiguo DECIMAL(10,2);
 DECLARE precioNuevo DECIMAL(10,2);
 DECLARE clienteMasActivo INT;
 DECLARE pedidoRegalo INT;
 
 -- manejo errores
DECLARE exit handler FOR SQLEXCEPTION 
BEGIN
	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'ERROR';
END ;

START TRANSACTION ;

SELECT productos.precio into precioAntiguo FROM productos
WHERE productos.id = productoId;

SET precioNuevo = precioAntiguo *0.85;

if precioNuevo < 5 then
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = ' El precio promocionado no puede ser inferior a 5';
END if;

UPDATE productos 
SET productos.precio = precioNuevo
WHERE productos.id = productoId;

 SELECT pedidos.clienteId INTO clienteMasActivo FROM pedidos
    WHERE pedidos.fechaRealizacion >= CURDATE() - INTERVAL 1 YEAR
    GROUP BY pedidos.clienteId
    ORDER BY COUNT(*) DESC
    LIMIT 1;

INSERT INTO pedidos(fechaRealizacion, fechaEnvio, direccionEntrega, comentarios, clienteId, empleadoId)
VALUES(CURDATE(), fechaEnvio, direccionEntrega, 'Regalo', clienteMasActivo, empleadoId);

SET pedidoRegalo = LAST_INSERT_ID();

INSERT INTO lineaspedido(pedidoId, productoId, unidades, precio)
VALUES(pedidoRegalo, productoId, 1, 0);

COMMIT ;
 
END //
 
DELIMITER ;
```
4. Trigger. (2 puntos)
 Incluya su solución en el fichero 4.solucionTrigger.sql.
 Cree un trigger llamado t_restringir_pedidos_digitales_nocturnos que impida realizar pedidos que
 incluyan productos digitales si el pedido se 
realiza entre las 0:00 y las 6:00 de la madrugada.
```sql
DELIMITER //

CREATE TRIGGER t_restringir_pedidos_digitales_nocturnos
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN

DECLARE fechaRealizacion DATE;
DECLARE tipoProducto VARCHAR(255);

SELECT HOUR(pedidos.fechaRealizacion) INTO fechaRealizacion FROM pedidos
WHERE pedidos.id = NEW.pedidoId;

SELECT tiposproducto.nombre into tipoProducto FROM tiposproducto
JOIN productos ON productos.tipoProductoId = tiposproducto.id
JOIN lineaspedido ON productos.id = lineaspedido.productoId
JOIN pedidos ON lineaspedido.pedidoId = pedidos.id
WHERE pedidos.id = NEW.pedidoId;

if tipoProducto = 'Digitales' AND (fechaRealizacion > 0 AND fechaRealizacion < 6)  then 
SIGNAL SQLSTATE '45000'
	SET MESSAGE_TEXT = 'NO SE PUEDEN REALIZAR PEDIDOS DE PRODUCTOS DIGITALES DESDE LAS 00:00 HASTA LAS 06:00';
END IF;
END //

DELIMITER ;
 ```
5. Función. (1,5 puntos)
Incluya su solución en el fichero 5.solucionFuncion.sql.

Cree una función llamada numero_productos_cliente que reciba como parámetro el ID de un cliente y devuelva el número total de productos diferentes que ese cliente ha comprado en todos sus pedidos.

Solo deben contarse productos distintos, aunque se hayan comprado varias veces.

Si el cliente no ha hecho ningún pedido, la función debe devolver 0.

```sql
DELIMITER //

 CREATE OR REPLACE FUNCTION numero_productos_cliente(clienteId INT) RETURNS INT
 BEGIN
 
 DECLARE res INT;
 
    SELECT IFNULL(COUNT(DISTINCT lineaspedido.productoId),0) INTO res FROM lineaspedido
    JOIN pedidos ON lineaspedido.pedidoId = pedidoId
    WHERE clienteId = pedidos.clienteId;
	 
	 RETURN res;
  
 END //
 DELIMITER ;
```

6. Vista. (1,5 puntos)
Incluya su solución en el fichero 6.solucionVista.sql.

Cree una vista llamada resumen_clientes_activos que muestre, para cada cliente que haya realizado al menos un pedido en el último año:

Nombre del cliente

Número total de pedidos realizados en el último año

Número total de productos comprados

Importe total gastado (precioUnitario × unidades)

```sql
-- Crear la vista
CREATE OR REPLACE VIEW resumen_clientes_activos AS
SELECT 
    c.nombre AS nombreCliente,
    COUNT(DISTINCT p.id) AS numeroPedidos,
    SUM(lp.unidades) AS totalProductos,
    SUM(lp.precioUnitario * lp.unidades) AS totalGastado
FROM Clientes c
JOIN Pedidos p ON c.id = p.idCliente
JOIN LineasPedido lp ON p.id = lp.idPedido
WHERE p.fechaRealizacion >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY c.id;

-- Consulta de ejemplo que utiliza la vista
-- Mostrar clientes que han gastado más de 500€ en el último año
SELECT *
FROM resumen_clientes_activos
WHERE totalGastado > 500;
```
