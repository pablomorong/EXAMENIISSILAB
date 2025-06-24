 /* 1.(2)- Implemente los requisitos proporcionados (RI/RN) en 
 MariaDB. No es necesario usar triggers. */
 
 ```sql
 CREATE TABLE eventos(
 id INT AUTO_INCREMENT PRIMARY KEY NOT NULL,
 usuarioId INT NOT NULL,
 nombre VARCHAR(255) NOT NULL,
 capacidad INT NOT NULL CHECK(capacidad >= 2 AND capacidad <= 100),
 fecha DATE NOT NULL CHECK(YEAR(fecha)> 2000),
 UNIQUE(usuarioId, fecha),
 FOREIGN KEY (usuarioId) REFERENCES users(idUser)
 	ON DELETE CASCADE
 	ON UPDATE CASCADE
);
```

/*2.(1)- Use Insert para implementar las pruebas de aceptación. 
Incluya captura de pantalla con los datos de la tabla después de 
las pruebas. */

```sql

INSERT INTO eventos(usuarioId, nombre, capacidad,fecha)
VALUES(2, 'Evento 1', 10, '2021-04-01'),
(2, 'Evento 2', 10, '2018-06-05'),
(1, 'Evento 3', 100, '2020-02-04');

/*Use Delete para eliminar el elemento con ID=1. Incluya captura 
de pantalla con los datos de la tabla después de la llamada. */
DELETE FROM eventos
WHERE eventos.id = 1;

UPDATE eventos 
SET capacidad = 30
WHERE id = 3;
```

/*3.(0.5)- Escriba y realice peticiones HTTP para obtener el 
listado de géneros de la base de datos. */
SELECT * FROM genres;

/*Cree una consulta que devuelva el precio medio de los 
videojuegos de plataforma con ID=2:*/
```sql
SELECT AVG(videogames.price) FROM videogames
WHERE videogames.idPlatform = 2;
```

/*5.(1.5)- Cree un disparador que, antes de actualizarse un 
Evento, si la capacidad está por encima de la media existente 
en la tabla, la establezca como 1000 en su lugar. Ejecute Update
con un valor de 5000 para comprobar que funciona el disparador. 
Pruebe el disparador. Incluya captura de pantalla con los 
datos de la tabla después de ejecutar la llamada. */

```sql
DELIMITER //
CREATE TRIGGER ejercicio5

BEFORE UPDATE ON eventos
FOR EACH ROW
BEGIN

DECLARE res DECIMAL(10,2);


SELECT AVG(eventos.capacidad) INTO res FROM eventos;



    IF res IS NOT NULL AND NEW.capacidad >  res THEN
        SET new.capacidad = 1000;
    END IF;
END //
DELIMITER ;

UPDATE eventos
SET capacidad = 5000
WHERE eventos.id = 2;
```

/*6.(0.5)- Cree una función que, a partir del identificador de 
un Evento, devuelva la cantidad de eventos de la tabla con una 
capacidad menor que la de dicho Evento. Use la función para 
obtener el resultado a partir del elemento con ID=2. Incluya 
capturas de pantallas. */

```sql
DELIMITER //
 CREATE OR REPLACE FUNCTION ejercicio6(eventoId INT) RETURNS DOUBLE
 BEGIN
  RETURN (
    SELECT COUNT(*) FROM eventos 
	 WHERE eventoId = eventos.id
  );
 END //
 DELIMITER ;
 
 SELECT ejercicio6(2);
``` 
 
 /*(0.5)- Cree un procedimiento que, a partir del identificador 
 de un Evento, lo borre solo si el resultado de la función 
 anterior es mayor a 1. */
 
 ```sql
DELIMITER //
CREATE OR REPLACE PROCEDURE ejericicio6(eventoId INT )
BEGIN

DECLARE cantidad INT;

SET cantidad = ejercicio6(eventoId);

if cantidad > 1 then 
DELETE FROM eventos WHERE eventos.id = eventoId;

END if;

END //
DELIMITER ;
```

/*7.(1)- Cree una consulta que devuelva el precio medio de los 
videojuegos de cada plataforma, ordenados por su precio medio 
de mayor a menor: */

```sql
SELECT AVG(videogames.price) as average FROM videogames
GROUP BY videogames.idPlatform 
ORDER BY videogames.price DESC;
```
/*8.(1)- Implemente las pruebas de aceptación como peticiones 
POST HTTP. Recuerde limpiar los datos de la tabla insertados los
 apartados anteriores. Las peticiones POST se pueden realizar 
 con o sin autenticación. Incluya capturas de pantallas. */
 
/*9.(0.5)- Cree una consulta que devuelva la descripción de los 
géneros con videojuegos valorados de media en más de 8, así como
 esta media:*/
 
 ```sql
 SELECT genres.`description`, AVG(videogames.score) AS average FROM genres
 JOIN genresvideogames ON genresvideogames.idGenre = genres.idGenre
 JOIN videogames ON genresvideogames.idVideogame = videogames.idVideogame
 GROUP BY genres.`description`
 HAVING average > 8
 ORDER BY average DESC ;
 ```
 
 10.(0.5)- Cree un procedimiento pAddTwoEvents(…) que, dentro 
 de una transacción, inserte dos ventos a partir de los datos 
 suministrados de los dos. 

Realice dos llamadas: una que inserte dos correctamente, y una 
en la que el segundo rompa alguna restricción y aborte la 
transacción. Incluya capturas de pantallas 

```sql
DROP PROCEDURE IF EXISTS pAddTwoEvents;

DELIMITER //

CREATE OR REPLACE PROCEDURE pAddTwoEvents(
   usuario1 INT, nombre1 VARCHAR(255), capacidad1 INT, fecha1 DATE,
   usuario2 INT, nombre2 VARCHAR(255), capacidad2 INT, fecha2 DATE
)
BEGIN 
-- manejo de errores
	DECLARE exit handler FOR SQLEXCEPTION 
BEGIN
	
	
	ROLLBACK;
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error al bonificar pedido retrasado';
END ;

    START TRANSACTION;

    INSERT INTO eventos(usuario, nombre, capacidad, fecha) VALUES(usuario1, nombre1, capacidad1, fecha1);
	INSERT INTO eventos(usuario, nombre, capacidad, fecha) VALUES(usuario2, nombre2, capacidad2, fecha2);
	
    COMMIT;
END //

DELIMITER ;

CALL pAddTwoEvents(
    5, 'Evento C', 60, '2025-07-03',
    7, 'Evento D', 99, '2025-07-04'
);

```