Añada el requisito de información Alumno Interno. Un alumno interno es un estudiante que colabora con un Departamento en actividades docentes o de investigación. Sus atributos son: el departamento en el que el estudiante participa como alumno interno, el estudiante involucrado, el año académico en el que se hace la colaboración y el número de meses que dura la colaboración. Hay que tener en cuenta las siguientes restricciones:
-	Los estudiantes sólo pueden ser alumnos internos una vez en un único curso académico.
-	El número de meses de la colaboración debe ser como máximo de 9 meses y como mínimo de 3.
-	Todos los atributos son obligatorios, menos el número de meses de la colaboración.

```sql
DROP TABLE if EXISTS alumnosinternos;

CREATE TABLE AlumnosInternos(
id INT PRIMARY KEY AUTO_INCREMENT,
departamentoId int NOT NULL,
alumnoId INT NOT NULL,
anyo INT NOT NULL,
meses INT CHECK (meses BETWEEN 3 AND 9),
UNIQUE (alumnoId,anyo),
FOREIGN KEY (alumnoId) REFERENCES students(studentId)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (departamentoId) REFERENCES departments(departmentId)
ON DELETE CASCADE
ON UPDATE CASCADE
);
```

Cree un procedimiento almacenado llamado pInsertInterns () que cree los siguientes alumnos internos:
-	Alumno interno del estudiante con ID=1, en el departamento con ID=1, en el año académico 2019, con una duración de 3 meses.
-	Alumno interno del estudiante con ID=1, en el departamento con ID=1, en el año académico 2020, con una duración de 6 meses.
-	Alumno interno del estudiante con ID=2, en el departamento con ID=1, en el año académico 2019.


```sql
delimiter //
CREATE OR REPLACE PROCEDURE pInsertInterns()
BEGIN 
INSERT INTO alumnosinternos(alumnoId,departamentoId,anyo,meses)
VALUES (1,1,2019,3), (1,1,2020,6), (2,1,2019,null);
END // 
delimiter ;
CALL pInsertInterns();
```
Cree un procedimiento almacenado llamado pUpdateInterns(s, d) que actualiza la duración de los alumnos internos correspondientes al estudiante con ID=s con el valor d. Ejecute la llamada a pUpdateInterns(1,9)

```sql
delimiter //
CREATE OR REPLACE procedure pUpdateInterns(s INT, d int)
BEGIN 
UPDATE alumnosinternos a
SET a.meses = d
WHERE a.id = s;
END // 


delimiter ;
```
Cree un procedimiento almacenado llamado pDeleteInterns(s) que elimina los alumnos internos correspondientes al estudiante con ID=s. Ejecute la llamada pDeleteInterns(2)

```sql
CALL pUpdateInterns(1, 9)

delimiter //
CREATE OR REPLACE PROCEDURE pDeleteInterns(s INT)
BEGIN
DELETE FROM alumnosinternos
WHERE id = s;
END //
delimiter ;

CALL pDeleteInterns(2);
``` 

Cree una consulta que devuelva el nombre del profesor, el nombre del grupo, y los créditos que imparte en él para todas las imparticiones de asignaturas por profesores. Un ejemplo de resultado de esta consulta es el siguiente:
![image](https://github.com/user-attachments/assets/88bb5f61-640c-4cc4-9acc-ae54d488679f)

```sql
SELECT professors.firstName, `groups`.`name`, teachingLoads.credits
FROM professors JOIN teachingLoads ON(professors.professorId = teachingloads.professorId)
JOIN `groups` ON(teachingloads.groupId = `groups`.groupId);
```
Cree una consulta que devuelva las tutorías con al menos una cita. Un ejemplo de resultado de la consulta anterior es el siguiente:
![image](https://github.com/user-attachments/assets/4ca151ec-5ee0-49b4-bd67-011360f7404e)

```sql
SELECT th.tutoringHoursId FROM tutoringhours th
JOIN appointments ON(th.tutoringHoursId = appointments.tutoringHoursId);
```
Cree una consulta que devuelva el nombre y apellidos de los profesores con un despacho en la planta 0. Un ejemplo de resultado de la consulta anterior es el siguiente:
![image](https://github.com/user-attachments/assets/ea06df7b-eb8d-45f9-ba51-726de43a08e3)

```sql
SELECT p.firstName, p.surname FROM professors p
JOIN offices o ON (p.officeId = o.officeId)
WHERE o.`floor` = 0;
```
Cree una consulta que devuelva, por cada método de acceso, la media de las notas suspensas, ordenados por esta última de menor a mayor 
(no tienen que aparecer los métodos de acceso que no se den en ningún alumno o nota). Un ejemplo de resultado de la consulta anterior es el siguiente:
![image](https://github.com/user-attachments/assets/582a24a7-e7f5-426b-884b-3152b527b3d3)

```sql
SELECT students.accessMethod AS metodo, AVG(grades.`value`) AS avgGrade FROM students
JOIN grades ON(grades.studentId = students.studentid)
WHERE grades.`value` < 5
GROUP BY metodo
ORDER BY avgGrade ASC;
```
Cree una consulta que devuelva el nombre y los apellidos de los dos estudiantes con mayor nota media, sus notas medias, y sus notas más baja. Un ejemplo de resultado de la consulta anterior es el siguiente:
![image](https://github.com/user-attachments/assets/1d03c67e-f80d-4552-90a5-fe4215742baa)

```sql
SELECT students.firstName,students.surname,AVG(grades.`value`) AS avgGrade, MIN(grades.`value`) AS minGrade FROM students
JOIN grades ON(grades.studentId = students.studentid)
GROUP BY students.firstName
ORDER by avgGrade DESC
LIMIT 2;
```
