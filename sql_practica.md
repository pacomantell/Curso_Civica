# Práctica SQL

Vamos a cear la base de datos de una cadena de supermecados que registra sus importes de ventas con los sigueintes datos:
- Fecha
- País
- Tipo de Tarjeta de crédito
- Categoría
- Importe de la venta

Tras crear los mapas conceptual y lógico, comenzamos a definir el modelo físico, desde las tablas de dimensión hasta la tabla de hechos:

 - Tablas de dimensión:
```sql
CREATE TABLE d_categoria(
id_categoria INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
desc_categoria VARCHAR(50) DEFAULT NULL);

CREATE TABLE d_pais(
id_pais INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
desc_pais VARCHAR(50) DEFAULT NULL);

CREATE TABLE d_tipo_tarjeta(
id_tipo_tarjeta INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
desc_tipo_tarjeta VARCHAR(50) DEFAULT NULL);

CREATE TABLE d_fecha(
id_fecha INT PRIMARY KEY NOT NULL DEFAULT 0,
anio INT NOT NULL,
mes INT NOT NULL,
desc_mes VARCHAR(50) NOT NULL,
dia INT NOT NULL,
fecha DATE DEFAULT NULL,
dia_semana VARCHAR(50) DEFAULT NULL);
 ```

 - Tabla de hechos
  ```sql
  CREATE TABLE h_ventas_usuario(
  id_pais INT FOREIGN KEY REFERENCES d_pais(id_pais),
  id_categoria INT FOREIGN KEY REFERENCES d_categoria(id_categoria),
  id_tipo_tarjeta INT FOREIGN KEY REFERENCES d_tipo_tarjeta(id_tipo_tarjeta),
  id_fecha INT FOREIGN KEY REFERENCES d_fecha(id_fecha),
  ventas DECIMAL(8,2));
  ```

Agregamos los registros mediante los ```INSERT``` proporcionados, procediendo primero con las tablas de dimensión y, por último, con la tabla de hechos.

## Cuestiones

1. Listar los dos componentes (id_categoria, desc_categoria) de la tabla de categorías que tiene esta empresa.
  ```sql
  SELECT id_categoria, desc_categoria FROM d_categoria;
  ```

2. Listar los diferentes registros de años que hay en la tabla de fechas. 
  ```sql
  SELECT DISTINCT anio FROM d_fecha;
  ```

3. Mostrar el importe total de ventas de la empresa. Utilizando la función agregada correspondiente.
  ```sql
  SELECT SUM(ventas) AS Total_Ventas FROM h_ventas;
  ```

4. Mostrar el importe total de ventas de la empresa en función de la categoría (id_categoria).
```sql
SELECT SUM(ventas) AS Total_Ventas,
id_categoria
FROM h_ventas
GROUP BY id_categoria
ORDER BY SUM(ventas) DESC;
```

5. Mostrar las ventas de la empresa en función de la desc_categoria cruzando con la tabla d_categoria.
```sql
SELECT SUM(ventas) AS Total_Ventas,
desc_categoria AS Categoria
FROM h_ventas A
INNER JOIN d_categoria B
ON A.id_categoria=B.id_categoria
GROUP BY desc_categoria
ORDER BY SUM(ventas) DESC;
```

6. ¿En que años se hacen más ventas?
```sql
SELECT SUM(ventas) AS Total_Ventas,
anio
FROM h_ventas A
INNER JOIN d_fecha B
ON A.id_fecha=B.id_fecha
GROUP BY anio
ORDER BY SUM(ventas) DESC;
```

7. ¿En que país se hacen menos ventas? ¿Y en cual más ventas?
```sql
--Tabla auxiliar 1
WITH Max_ventas_CTE (Max_Ventas, Max_Pais) AS
(
--Guardo el maximo
SELECT TOP 1 SUM(ventas) AS Max_Ventas,
desc_pais
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
GROUP BY desc_pais
ORDER BY SUM(ventas) DESC
)
,
--Tabla auxiliar 2
Min_ventas_CTE (Min_Ventas, Min_Pais) AS
(
--Guardo el minimo
SELECT TOP 1 SUM(ventas) AS Max_Ventas,
desc_pais
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
GROUP BY desc_pais
ORDER BY SUM(ventas) ASC
)
--Selecciono los dos registros de las tablas auxiliares
SELECT Max_Ventas, Max_Pais
FROM Max_ventas_CTE
UNION
SELECT Min_Ventas, Min_Pais
FROM Min_ventas_CTE
ORDER BY Max_Ventas DESC;
```

8. Listar los 10 primeros países con más ventas. Utilizando la consulta de predicado correspondiente.
```SQL
SELECT TOP 10 SUM(ventas) AS Total_Ventas,
desc_pais
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
GROUP BY desc_pais
ORDER BY SUM(ventas) DESC;
```

9. Listar todas las ventas por año en el país de España. Utilizando la cláusula correspondiente.
```SQL
SELECT SUM(ventas) AS Total_Ventas_Spain,
anio
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
INNER JOIN d_fecha C
ON A.id_fecha=C.id_fecha
WHERE desc_pais='Spain'
GROUP BY anio;
```

10. ¿Con que tipo de tarjeta compra más la gente de Francia?
```sql
SELECT TOP 1 SUM(ventas) AS Total_Ventas,
desc_tipo_tarjeta
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
INNER JOIN d_tipo_tarjeta C
ON A.id_tipo_tarjeta=C.id_tipo_tarjeta
WHERE desc_pais='France'
GROUP BY desc_tipo_tarjeta;
```

11. Crear una vista con el número de ventas y los diferentes conceptos de negocio con lo que hemos trabajado: categoría, país, tipo 
de tarjeta, fecha.
```sql
CREATE VIEW Ventas_View AS
SELECT ventas,
desc_categoria AS Categoria,
desc_pais AS Pais,
desc_tipo_tarjeta AS Tarjeta,
fecha AS Fecha
FROM h_ventas A
INNER JOIN d_categoria B
ON A.id_categoria=B.id_categoria
INNER JOIN d_pais C
ON A.id_pais=C.id_pais
INNER JOIN d_tipo_tarjeta D
ON A.id_tipo_tarjeta=D.id_tipo_tarjeta
INNER JOIN d_fecha E
ON A.id_fecha=E.id_fecha;
```

12. Hacer una consultas de las ventas por países en el periodo de fechas de 21/12/2016 a 31/12/2016. Compararla con el mismo
periodo pero de 2015.
¿Qué año se vendió más en el mismo periodo?
```sql
--Ventas por paises de 21/12/2016 a 31/12/2016
SELECT SUM(ventas) AS ventas_2016,
Pais
FROM Ventas_View
WHERE Fecha BETWEEN '2016-12-21' AND '2016-12-31'
GROUP BY Pais
ORDER BY SUM(ventas) DESC;

--Ventas por paises de 21/12/2015 a 31/12/2015
SELECT SUM(ventas) AS ventas_2016,
Pais
FROM Ventas_View
WHERE Fecha BETWEEN '2015-12-21' AND '2015-12-31'
GROUP BY Pais
ORDER BY SUM(ventas) DESC;

--Comparacion total ventas en los dos periodos
SELECT SUM(ventas) AS ventas_2016,
(
SELECT SUM(ventas)
FROM Ventas_View
WHERE Fecha BETWEEN '2015-12-21' AND '2015-12-31'
) AS ventas_2015
FROM Ventas_View
WHERE Fecha BETWEEN '2016-12-21' AND '2016-12-31';
```
- Otra forma en una sola consulta
```sql
WITH ventas_2015 (venta, pais, id_pais) AS
(
SELECT SUM(ventas) AS venta_2015,
Pais,
id_pais
FROM Ventas_View A
INNER JOIN d_pais B
ON A.Pais=B.desc_pais
WHERE Fecha BETWEEN '2015-12-21' AND '2015-12-31'
GROUP BY Pais, id_pais
),
ventas_2016 (venta, pais, id_pais) AS
(
SELECT SUM(ventas) AS venta_2016,
Pais,
id_pais
FROM Ventas_View A
INNER JOIN d_pais B
ON A.Pais=B.desc_pais
WHERE Fecha BETWEEN '2016-12-21' AND '2016-12-31'
GROUP BY Pais, id_pais
)
SELECT A.venta AS venta_2015,
A.pais,
B.venta AS venta_2016,
B.Pais
FROM ventas_2015 A
FULL JOIN ventas_2016 B
ON A.id_pais=B.id_pais
GROUP BY A.venta, A.pais, B.venta, B.pais

UNION

SELECT SUM(ventas) AS ventas_2015,
CONVERT(varchar, YEAR(Fecha)) AS anio1,
(
SELECT SUM(ventas)
FROM Ventas_View
WHERE Fecha BETWEEN '2016-12-21' AND '2016-12-31'
) AS ventas_2015,
CONVERT( VARCHAR,(
SELECT TOP 1 YEAR(Fecha)
FROM Ventas_View
WHERE Fecha BETWEEN '2016-12-21' AND '2016-12-31'
)) AS anio2
FROM Ventas_View
WHERE Fecha BETWEEN '2015-12-21' AND '2015-12-31'
GROUP BY YEAR(Fecha);
```

13. Sacar un listado de ventas de la vista creada, que muestre las ventas para el año 2015 y enero. (usar las funciones de fecha de
SQL Server).
```sql
SELECT SUM(ventas) AS Ventas_enero_2015
FROM Ventas_View
WHERE MONTH(Fecha)=1
AND YEAR(Fecha)=2015;
```

14. Listar los países que facturen un importe de ventas total de más de 50.000 en alguno de los dos años.
```sql
SELECT SUM(ventas) AS Ventas,
Pais,
YEAR(Fecha) AS Anio
FROM Ventas_View
WHERE YEAR(Fecha)=2015
OR YEAR(Fecha)=2016
GROUP BY Pais, YEAR(Fecha)
HAVING SUM(ventas)>50000;
```

15. Listar los países que facturen un importe de ventas total de más de 10.000 en el mes de febrero de 2016.
```sql
SELECT SUM(ventas) AS Ventas,
Pais
FROM Ventas_View
WHERE MONTH(Fecha)=2
AND YEAR(Fecha)=2016
GROUP BY Pais
HAVING SUM(ventas)>10000;
```

16. La media de ventas de España. 
```sql
SELECT AVG(ventas) AS media_ventas_esp
FROM Ventas_View
WHERE Pais='Spain';
```

17. Contar el número de países que en enero de 2016 tienen un total de ventas mayor 100. Hacer la consulta sobre la tabla creada de ventas_negocio.
```sql
SELECT COUNT(*) AS num_paises
FROM
(
SELECT SUM(ventas) AS venta,
Pais
FROM Ventas_View
WHERE MONTH(Fecha)=1
AND YEAR(Fecha)=2016
group by Pais
HAVING SUM(ventas)>100) AS listado;
```

18. Obtener la venta máxima de enero, en la categoría Clothing, para los dos años (2015, 2016) ¿Que año se hizo la máxima venta más alta?
```sql
SELECT MAX(ventas) AS venta_max,
YEAR(Fecha) AS anio
FROM Ventas_View
WHERE Categoria='Clothing'
AND MONTH(Fecha)=1
GROUP BY YEAR(Fecha);
```

19. Mostrar los países cuyas descripciones empiecen por la letra S.
```sql
SELECT desc_pais
FROM d_pais
WHERE desc_pais LIKE'S%';
```

20. Mostrar los países cuyas descripciones empiecen por la letra S y terminen con la letra n.
```sql
SELECT desc_pais
FROM d_pais
WHERE desc_pais LIKE'S%n';
```

21. La tarjeta más utilizada por cada país (es decir que se haga más número de ventas no mayor importe).
```sql
SELECT Tarjeta,
num_op,
Pais
FROM
(
SELECT Tarjeta,
COUNT(Tarjeta) as num_op,
Pais,
RANK() OVER (PARTITION BY Pais ORDER BY COUNT(Tarjeta) DESC) AS venta_rank
FROM Ventas_View
GROUP BY Pais, Tarjeta
) AS max_ventas
WHERE venta_rank=1
GROUP BY Pais, Tarjeta, num_op
ORDER BY Pais
```

22. Mes en el que se gasta más dinero, del año 2015.
```sql
SELECT TOP 1 MONTH(Fecha) AS Mes,
SUM(Ventas) AS total_ventas
FROM Ventas_View
WHERE YEAR(Fecha)=2015
GROUP BY MONTH(Fecha)
ORDER BY SUM(Ventas) DESC
```

23. Mes en el que se gasta más dinero por país, del año 2015.
```sql
SELECT Mes AS mes,
Pais,
total_venta
FROM
(
SELECT MONTH(Fecha) AS Mes,
Pais,
SUM(ventas) AS total_venta,
RANK() OVER (PARTITION BY Pais ORDER BY SUM(Ventas) DESC) AS venta_rank
FROM Ventas_View
WHERE YEAR(Fecha)=2015
GROUP BY MONTH(Fecha), Pais
) AS max_ventas
WHERE venta_rank=1
GROUP BY Mes, Pais, total_venta
ORDER BY Pais
```
