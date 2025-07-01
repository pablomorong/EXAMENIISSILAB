### 1. Creación de tabla. (1,5 puntos)

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.

Necesitamos conocer la garantía de nuestros productos. Para ello se propone la creación de una nueva tabla llamada `Garantias`. Cada producto tendrá como máximo una garantía (no todos los productos tienen garantía), y cada garantía estará relacionada con un producto.

Para cada garantía necesitamos conocer la fecha de inicio de la garantía, la fecha de fin de la garantía, si tiene garantía extendida o no.

Asegure que la fecha de fin de la garantía es posterior a la fecha de inicio.

```sql
CREATE TABLE garantias(
garantiaId INT PRIMARY KEY AUTO_INCREMENT,
productoId INT,
fechaInicio DATE,
fechaFin DATE CHECK(fechaFin>fechaInicio),
garantiaExtendida BOOLEAN,
UNIQUE(productoId),
FOREIGN KEY (productoId) REFERENCES productos(id)
	ON DELETE CASCADE
	ON UPDATE CASCADE
); 
```

### 2. Consultas SQL (DQL). (3 puntos)

Incluya su solución en el fichero `2.solucionConsultas.sql`.

#### 2.1. Devuelva el nombre del producto, nombre del tipo de producto, y precio unitario al que se vendieron los productos digitales (1 punto)

```sql
SELECT productos.nombre, tiposproducto.nombre, lineaspedido.precio FROM lineaspedido
JOIN productos ON productos.id = lineaspedido.productoId
JOIN tiposproducto ON productos.tipoProductoId = tiposproducto.id
WHERE tiposproducto.nombre = 'Digitales';
```

#### 2.2. Consulta que devuelva el nombre del empleado, el número de pedidos de más de 500 euros gestionados en este año y el importe total de cada uno de ellos, ordenados de mayor a menor importe gestionado. Los empleados que no hayan gestionado ningún pedido, también deben aparecer. (2 puntos)

```sql
SELECT usuarios.nombre, COUNT(DISTINCT pedidos.id) AS numPedidos, SUM(lineaspedido.precio * lineaspedido.unidades) AS importeTotal FROM pedidos
LEFT JOIN lineaspedido ON lineaspedido.pedidoId = pedidos.id
JOIN empleados ON pedidos.empleadoId = empleados.id
JOIN usuarios ON usuarios.id = empleados.usuarioId
GROUP BY usuarios.nombre
HAVING importeTotal > 500
ORDER BY importeTotal;
``` 


### 3. Procedimiento. Actualizar precio de un producto y líneas de pedido no enviadas. (3,5 puntos)

Incluya su solución en el fichero `3.solucionProcedimiento.sql`.

Cree un procedimiento que permita actualizar el precio de un producto dado y que modifique los precios de las líneas de pedido asociadas al producto dado solo en aquellos pedidos que aún no hayan sido enviados. (1,5 puntos)

Asegure que el nuevo precio no sea un 50% menor que el precio actual y lance excepción si se da el caso con el siguiente mensaje: (1 punto)

`No se permite rebajar el precio más del 50%`.

Garantice que o bien se realizan todas las operaciones o bien no se realice ninguna. (1 punto)

```sql 

DELIMITER //
CREATE OR REPLACE PROCEDURE ejercicio3(productoId INT , nuevoPrecio DECIMAL(10,2))

BEGIN 

DECLARE precioActual DECIMAL(10,2);

-- manejo errores
DECLARE exit handler FOR SQLEXCEPTION 
BEGIN
	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'ERROR';
END ;

START TRANSACTION ;

SELECT productos.precio into precioActual FROM productos
WHERE productos.id = productoId;

if nuevoPrecio < precioActual * 0.5 then
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se permite rebajar el precio más del 50%';
END if;


UPDATE productos
SET productos.precio = nuevoPrecio
WHERE productoId = productos.id;

UPDATE lineaspedido
SET lineaspedido.precio = nuevoPrecio
WHERE lineaspedido.productoId = productoId AND pedidoId IN (SELECT id FROM Pedidos WHERE fechaEnvio IS NULL); -- pedidos no enviados


COMMIT;


END //

DELIMITER ;

```


### 4. Trigger. 2 puntos.

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `t_asegurar_mismo_tipo_producto_en_pedidos` que impida que, a partir de ahora, un mismo pedido incluya productos físicos y digitales.

```sql

DELIMITER //

CREATE TRIGGER t_asegurar_mismo_tipo_producto_en_pedidos
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN

DECLARE IDTipoProductoActual INT;
DECLARE v_existeOtroTipoProductoEnPedido BOOLEAN;

SELECT productos.tipoProductoId INTO IDTipoProductoActual FROM productos
WHERE productos.id = NEW.productoId;

SELECT EXISTS(
SELECT * FROM lineaspedido
JOIN productos ON lineaspedido.productoId = productos.id
WHERE lineaspedido.pedidoId = NEW.pedidoId AND p.tipoProductoId <> v_idTipoProductoNuevo
) INTO v_existeOtroTipoProductoEnPedido;

if v_existeOtroTipoProductoEnPedido then 
SIGNAL SQLSTATE '45000'
	SET MESSAGE_TEXT = 'El pedido no puede incluir productos de tipos diferentes (físicos y digitales).';
END IF;
END //

DELIMITER ;
```
