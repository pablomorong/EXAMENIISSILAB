# Enunciado Evaluación Individual de Laboratorio - Modelo C

**Si usted entrega sin haber sido verificada su identidad no podrá ser evaluado.**

## Tienda Online Avanzada

Partiendo de la base de datos `tiendaOnline` definida con las tablas:  
- Usuarios, Empleados, Clientes, Pedidos, TiposProducto, Productos, LineasPedido  
(Esquema proporcionado en el script SQL adjunto)

---

### 1. Creación de tabla. (1,5 puntos)

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.

Se requiere almacenar información sobre **Devoluciones** de productos. Cada línea de pedido podrá tener como máximo una devolución. La tabla `Devoluciones` debe incluir:  
- Identificador (PK)  
- Línea de pedido (FK)  
- Fecha de devolución  
- Motivo de la devolución (texto)  
- Estado de la devolución (`Pendiente`, `Aceptada`, `Rechazada`) - tipo ENUM o VARCHAR con restricción  

Asegure que la fecha de devolución no sea anterior a la fecha de realización del pedido correspondiente.

---

```sql
   CREATE TABLE devoluciones(
 id INT PRIMARY KEY AUTO_INCREMENT,
 lineaspedidoId INT,
 fechaDevolucion DATE,
 motivo TEXT,
 estado ENUM('Pendiente','Aceptada','Rechazada'),
 FOREIGN KEY (lineaspedidoId) REFERENCES lineaspedido(id)
 	ON DELETE CASCADE
 	ON UPDATE CASCADE
 );
```

### 2. Consultas SQL (DQL). (3 puntos)

Incluya su solución en el fichero `2.solucionConsultas.sql`.

#### 2.1. Devuelva el nombre del producto, el nombre del tipo de producto y el precio unitario con el que se vendieron los productos cuyo precio fue mayor a 100 euros. (1 punto)

```sql  
SELECT productos.nombre, tiposproducto.nombre, lineaspedido.precio
FROM productos
JOIN tiposproducto ON tiposproducto.id = productos.tipoProductoId
JOIN lineaspedido ON lineaspedido.productoId = productos.id
WHERE lineaspedido.precio > 100
GROUP BY productos.nombre;
```

#### 2.2. Devuelva el nombre del empleado, el número de pedidos gestionados en los últimos 6 meses con importe total superior a 1000 euros, y el importe total gestionado por empleado. Incluya también empleados que no hayan gestionado pedidos. Ordene por importe total gestionado descendente. (2 puntos)

```sql
 SELECT usuarios.nombre, COUNT(DISTINCT pedidos.id), (lineaspedido.precio * lineaspedido.unidades) AS importeTotal FROM lineaspedido
 JOIN pedidos ON lineaspedido.pedidoId = pedidos.id
 RIGHT JOIN empleados ON empleados.id = pedidos.empleadoId
 RIGHT JOIN usuarios ON empleados.usuarioId = usuarios.id
 WHERE pedidos.fechaRealizacion >= CURDATE() - INTERVAL 6 MONTH
 GROUP BY empleados.id
 HAVING importeTotal > 1000
 ORDER BY importeTotal DESC;
 ```
---

### 3. Procedimiento. Actualizar precio y líneas pendientes. (3,5 puntos)

Incluya su solución en el fichero `3.solucionProcedimiento.sql`.

Cree un procedimiento que permita actualizar el precio de un producto dado y que modifique el precio en las líneas de pedido asociadas a ese producto solo en aquellos pedidos que **aún no hayan sido enviados** (`fechaEnvio IS NULL`).

- Asegure que el nuevo precio no sea inferior al 70% del precio actual.  
- En caso contrario, lance una excepción con el mensaje:  
  `La reducción de precio supera el 30% permitida.`  
- Garantice que las operaciones se realicen de forma atómica (transacción).


---

```sql 
DELIMITER //
 CREATE OR REPLACE PROCEDURE ejercicio3(productoId INT, precioNuevo DECIMAL(10,2))
 BEGIN 
 
 DECLARE precioActual DECIMAL(10,2);
 
 -- manejo errores
DECLARE exit handler FOR SQLEXCEPTION 
BEGIN
	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'LA TRANSACCIÓN HA TENIDO ALGÚN ERROR';
END ;

START TRANSACTION ;

SELECT productos.precio into precioActual FROM productos
WHERE productos.id = productoId;


 if precioNuevo < precioActual * 0.7 then 
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'La reducción de precio supera el 30% permitida';
END if;


 UPDATE productos
 SET productos.precio = precioNuevo
 WHERE productos.id = productoId;
 
 UPDATE lineaspedido
 SET lineaspedido.precio = precioNuevo
 WHERE lineaspedido.fechEnvio IS NULL 
 AND lineaspedido.productoId = productoId;
 
COMMIT ;

END //
 
DELIMITER ;

```

### 4. Trigger. (2 puntos)

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `t_no_mezclar_tipos_productos_en_pedido` que impida que un mismo pedido incluya productos de más de un tipo (es decir, todos los productos de un pedido deben pertenecer al mismo `tipoProductoId`). Lance error si se intenta insertar una línea que viola esta regla.

---
```sql
DELIMITER //
CREATE TRIGGER t_no_mezclar_tipos_productos_en_pedido
BEFORE INSERT ON lineaspedido
FOR EACH ROW
BEGIN

DECLARE tipoProductoId INT;
DECLARE existe BOOLEAN;


SELECT productos.tipoProductoId into tipoProductoId FROM productos
WHERE productos.id = NEW.productoId;

SELECT EXISTS (
SELECT * FROM lineaspedido
JOIN productos ON lineaspedido.productoId = productos.id
JOIN pedidos ON lineaspedido.pedidoId = pedidos.id
WHERE pedidos.id = NEW.pedidoId
AND tipoProductoId != productos.tipoProductoId) INTO existe;

IF existe THEN
     SIGNAL SQLSTATE '45000'
     SET MESSAGE_TEXT = 'todos los productos de un pedido deben pertenecer al mismo `tipoProductoId';
    END IF;
END //

DELIMITER ;
 
```

### 5. Función. (2 puntos)

Incluya su solución en el fichero `5.solucionFuncion.sql`.

Cree una función llamada `f_total_cliente` que reciba el `clienteId` y devuelva la suma total de dinero gastado en pedidos enviados (es decir, solo aquellos pedidos con `fechaEnvio` no nulo), considerando el precio y las unidades de las líneas de pedido asociadas.

---

```sql
DELIMITER //

 CREATE OR REPLACE FUNCTION ejercicio5(clienteId INT) RETURNS DOUBLE
 BEGIN
 
 DECLARE res DOUBLE;
 
    SELECT SUM(lineaspedido.precio * lineaspedido.unidades) into res FROM lineaspedido
    JOIN pedidos ON lineaspedido.pedidoId = pedidos.id
    WHERE pedidos.clienteId = clienteId 
	 AND pedidos.fechaEnvio IS NULL;
	 
	 RETURN res;
  
 END //
 DELIMITER ;
```

## Procedimiento de entrega:

### 1. Comprimir ficheros

Cree un fichero `zip` que incluya:

- `1.solucionCreacionTabla.sql`  
- `2.solucionConsultas.sql`  
- `3.solucionProcedimiento.sql`  
- `4.solucionTrigger.sql`  
- `5.solucionFuncion.sql`

### 2. Subir fichero `zip`

Súbalo en la plataforma correspondiente. **No pulse enviar antes de la verificación.**

### 3. Avisar a profesor ANTES de realizar la entrega

Muestre su DNI o documento para verificación.

**Sin verificación no se evaluará.**
