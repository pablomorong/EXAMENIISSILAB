•	Utilice HeidiSQL para abrir la conexión del root con contraseña iissi$root y:	
o	Cree un nuevo usuario cuyo nombre sea UVUS_USER. 
o	Cree una conexión para el nuevo usuario y ejecute el script SQL proporcionado.
•	Implemente la solución a los ejercicios planteados. Estos ejercicios están basados en unos requisitos adicionales (Catálogo de requisitos) que se suministran a continuación.
•	En cada ejercicio, incluya el código desarrollado (con formato de texto) y las capturas de pantalla que muestren el resultado de ejecutar dicho ejercicio.
•	Debe subir a la actividad de EV este documento con el código y las capturas de pantallas. Antes de subir la actividad, avise a un profesor para que valide la misma.

Catálogo de requisitos
RI-1-01	Requisito de información
Como:	Profesor de la asignatura
Quiero:	Tener disponible información sobre las valoraciones que cada usuario haya podido realizar para cada juego, teniendo en cuenta que un usuario puede valorar muchos juegos, y un juego puede ser valorado por muchos usuarios. Es necesario guardar información de la fecha de la valoración, la puntuación que ha otorgado cada usuario a cada juego, su opinión en forma de texto, el número de “likes” que ha recibido esa valoración por parte de otros usuarios y el veredicto final sobre el juego. Todos los atributos son obligatorios salvo el número de “likes”, que por defecto será 0.
Para:	Evaluar al estudiante

RN-1-01	Regla de negocio de la puntuación
Como:	Profesor de la asignatura
Quiero:	El usuario podrá otorgar a cada juego una puntuación entre 0 y 5, ambas inclusive.
Para:	Evaluar al estudiante

RN-1-02	Regla de negocio del veredicto
Como:	Profesor de la asignatura
Quiero:	El usuario asignará un veredicto final a su valoración sobre cada juego, este veredicto podrá ser ‘Imprescindible’, ‘Recomendado’, ‘Comprar en rebajas’, ‘No merece la pena’.
Para:	Evaluar al estudiante


RN-1-03	Regla de negocio de valoraciones
Como:	Profesor de la asignatura
Quiero:	Un usuario no podrá hacer más de una valoración sobre un juego determinado.
Para:	Evaluar al estudiante

Prueba1	Inserción de valoración de juegos
Como:	Profesor de la asignatura
Quiero:	 	Insertar el progreso del usuario 1 en el juego 2, puntuación = 5, un comentario de texto cualquiera, y veredicto ‘Imprescindible’, fecha de hoy.
 	Insertar el progreso del usuario 2 en el juego 4, puntuación = 3, un comentario de texto cualquiera, y estado ‘Comprar en rebajas’, fecha de hoy.
 	Insertar el progreso del usuario 3 en el juego 3, puntuación = 4, un comentario de texto cualquiera, y estado ‘Recomendado’, fecha de hoy.
 	Insertar el progreso del usuario 4 en el juego 5, puntuación = 1, un comentario de texto cualquiera, y estado ‘No merece la pena’, fecha de hoy.
 	Insertar el progreso del usuario 2 en el juego 3, puntuación = 4.5, un comentario de texto cualquiera, y estado ‘Imprescindible’, fecha de hoy.

 	Insertar el progreso del usuario 1 en el juego 6, puntuación = 10, un comentario de texto cualquiera, y estado ‘Imprescindible’. (RN-1-01)
 	Insertar el progreso del usuario 3 en el juego 1, puntuación = 3, un comentario de texto cualquiera, y estado ‘Ni fu ni fa’. (RN-1-02)
 	Insertar el progreso del usuario 3 en el juego 3, puntuación = 2, comentario ‘No era para tanto’, y estado ‘No merece la pena’. (RN-1-03)
 	Insertar el progreso del usuario 6 en el juego 8, puntuación = 3, un comentario de texto cualquiera, y estado ‘Comprar en rebajas’. (referencia no existente)

Para:	Evaluar al estudiante.


Ejercicio-1 (2 puntos) Implemente los requisitos proporcionados (RI/RN).
(La tabla puntúa 1 y cada restricción 0,5)

```sql
CREATE TABLE valoraciones(
	id INT AUTO_INCREMENT PRIMARY KEY,
	fechaValoracion DATE NOT NULL ,
	puntuacion DOUBLE NOT NULL CHECK (puntuacion>=0 AND puntuacion<=5) , 
	opinion TEXT NOT null, 
	numLikes INT DEFAULT 0,
	veredictoFinal ENUM ('Imprescindible', 'Recomendado', 'Comprar en rebajas', 'No merece la pena') NOT NULL,
	jugadorId INT NOT NULL, 
	videojuegoId INT NOT NULL, 
	UNIQUE(jugadorId, videojuegoId),
	FOREIGN KEY (jugadorId) REFERENCES jugadores(jugadorId)
		ON DELETE CASCADE
		ON UPDATE CASCADE,
	FOREIGN KEY (videojuegoId) REFERENCES videojuegos(videojuegoId)
		ON DELETE CASCADE
		ON UPDATE CASCADE
	);
```

Calificación: ______


Ejercicio-2 (2 puntos)
Codifique un procedimiento almacenado que inserte una nueva valoración de un usuario concreto para un juego dado en la tabla creada en el ejercicio anterior, que será llamado tantas veces como progresos se deseen añadir.
Para comprobar las RN y las restricciones de Integridad, se llamará al procedimiento con los parámetros que aparecen en Prueba1-Inserción de valoración de juegos. Las valoraciones en verde se insertarán y las marcadas en rojo serán rechazadas.
 (Si la inserción se realizar directamente con varios comandos INSERT de SQL, entonces se puntúa 0,5)
 ```sql
DELIMITER //
 CREATE OR REPLACE PROCEDURE pInsertarValoraciones(jugadorId INT, videojuegoId INT, puntuacion INT, opinion TEXT, veredictoFinal VARCHAR(255), fechaValoracion DATE)
 BEGIN
 	INSERT INTO valoraciones(jugadorId, videojuegoId, puntuacion, opinion, veredictoFinal, fechaValoracion)
 	VALUES (jugadorId, videojuegoId, puntuacion, opinion, veredictoFinal, fechaValoracion);

 END //
 DELIMITER ;
 
 CALL pInsertarValoraciones(1,2,5.0, 'GOTY', 'Imprescindible', CURDATE());
 CALL pInsertarValoraciones(2,4,3.0, 'Error', 'Comprar en rebajas', CURDATE());
 CALL pInsertarValoraciones(3,3,4.0, 'ANTIGOTY', 'Recomendado', CURDATE());
 CALL pInsertarValoraciones(4,5,1.0, 'malillo', 'No merece la pena', CURDATE());
 CALL pInsertarValoraciones(2,3,4.5, 'ANTIGOTY', 'Imprescindible', CURDATE());
```

Calificación: ______



Ejercicio-3 (1 puntos)
Cree una consulta que devuelva todos los usuarios, sus juegos y las valoraciones respectivas, ordenados por videojuegosId.
Un ejemplo de resultado de esta consulta es el siguiente:
 ```sql
SELECT jugadores.nickname,  videojuegos.*, valoraciones.*
 FROM valoraciones
 JOIN videojuegos ON valoraciones.videojuegoId = videojuegos.videojuegoId
 JOIN jugadores ON valoraciones.jugadorId = jugadores.jugadorId
 ORDER BY videojuegos.videojuegoId;
 ```


Calificación: ______



Ejercicio-4 (1 puntos)
Codifique un trigger para impedir que la fecha de una valoración sea anterior a la fecha de lanzamiento del juego, y posterior a la fecha actual. Añada una instrucción que haga saltar el trigger.

```sql
DELIMITER //
CREATE TRIGGER ejercicio4 

BEFORE INSERT ON valoraciones
FOR EACH ROW
BEGIN

DECLARE fl DATE;


SELECT videojuegos.fechaLanzamiento INTO fl
FROM videojuegos
WHERE NEW.videojuegoId = videojuegos.videojuegoId;


    IF NEW.fechaValoracion <  fl THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'El cliente debe tener al menos 18 años para comprar este producto.';
    END IF;
END //
DELIMITER ;
```


Calificación: ______



Ejercicio-5 (1 puntos)
Codifique una función que devuelva el número de valoraciones de un usuario dado.
Realice una prueba de la función con UsuarioId=2.

```sql
DELIMITER //
 CREATE OR REPLACE FUNCTION ejercicio4(usuarioId INT) RETURNS DOUBLE
 BEGIN
  RETURN (
    SELECT COUNT(*) FROM valoraciones 
	 WHERE usuarioId = valoraciones.jugadorId 
  );
 END //
 DELIMITER ;
```

Calificación: ______



Ejercicio-6 (1 puntos)
Cree una consulta que devuelva los juegos y la media de las valoraciones recibidas, ordenados de mayor a menor. En el listado deben aparecer todos los juegos, tengan o no valoración.
Un ejemplo de resultado de esta consulta es el siguiente:
 
 ```sql
SELECT videojuegos.nombre, AVG (valoraciones.puntuacion) AS mediaPuntuacion FROM valoraciones
 RIGHT JOIN  videojuegos ON valoraciones.videojuegoId = videojuegos.videojuegoId
 GROUP BY videojuegos.nombre
 ORDER BY mediaPuntuacion DESC;
 ```


Calificación: ______

Ejercicio-7 (1 puntos) 
Cree un trigger que impida valorar un juego que esté en fase ’Beta’. (1 punto)

```sql
DELIMITER //
 CREATE OR REPLACE TRIGGER ejercicio7
 BEFORE INSERT ON valoraciones FOR EACH ROW
 BEGIN
 
 DECLARE estado VARCHAR(255);
 
 SELECT videojuegos.estado INTO estado FROM videojuegos
 WHERE NEW.videojuegoId = videojuegos.videojuegoId;
 
 
 
 IF estado = 'Beta' THEN
 SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT =
 'No se puede valorar un juego en fase beta';
 END IF;
 END //
 DELIMITER ;

```