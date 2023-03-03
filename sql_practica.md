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
SELECT SUM(ventas)AS Total_Ventas,
id_categoria
FROM h_ventas
GROUP BY id_categoria
ORDER BY id_categoria;
```
5. Mostrar las ventas de la empresa en función de la desc_categoria cruzando con la tabla d_categoria.
```sql
SELECT SUM(ventas) AS Total_Ventas,
desc_categoria AS Categoria
FROM h_ventas A
INNER JOIN d_categoria B
ON A.id_categoria=B.id_categoria
GROUP BY desc_categoria
ORDER BY desc_categoria;
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
SELECT SUM(ventas) AS Total_Ventas,
desc_pais
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
GROUP BY desc_pais
ORDER BY SUM(ventas) DESC;
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
SELECT SUM(ventas) AS Total_Ventas,
desc_pais,
anio
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
INNER JOIN d_fecha C
ON A.id_fecha=C.id_fecha
WHERE desc_pais='Spain'
GROUP BY anio, desc_pais;
```
10. ¿Con que tipo de tarjeta compra más la gente de Francia?
```sql
SELECT Top 1 SUM(ventas) AS Total_Ventas,
desc_pais,
desc_tipo_tarjeta
FROM h_ventas A
INNER JOIN d_pais B
ON A.id_pais=B.id_pais
INNER JOIN d_tipo_tarjeta C
ON A.id_tipo_tarjeta=C.id_tipo_tarjeta
WHERE desc_pais='France'
GROUP BY desc_tipo_tarjeta, desc_pais;
```
11. Crear una vista con el número de ventas y los diferentes conceptos de negocio con lo que hemos trabajado: categoría, país, tipo 
de tarjeta, fecha.
