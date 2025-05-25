
# Proyecto SQL: Análisis de Ventas Retail

## Descripción

Este proyecto consiste en analizar un conjunto de datos de ventas minoristas utilizando SQL. El objetivo principal es demostrar habilidades en creación y manejo de bases de datos, limpieza de datos, análisis exploratorio y generación de insights a partir de consultas SQL.

La base de datos creada contiene información de transacciones con detalles como fecha, hora, cliente, categoría de producto, cantidad, costos y ventas totales.

## Estructura del Proyecto

### 1. Creación y configuración de la base de datos

- Se crea la base de datos `sql_project_p1`.
- Se crea la tabla `retail_sales` con las columnas necesarias para almacenar las ventas.

```sql
CREATE DATABASE IF NOT EXISTS sql_project_p1;
USE sql_project_p1;

DROP TABLE IF EXISTS retail_sales;
CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(15),
    age INT,
    category VARCHAR(15),
    quantity INT,
    price_per_unit FLOAT,
    cogs FLOAT,
    total_sale FLOAT
);
```

### 2. Limpieza de datos

- Se verifica la existencia de valores nulos en columnas importantes.
- Se eliminan los registros con valores nulos en campos clave para asegurar la calidad de los datos.

```sql
DELETE FROM retail_sales
WHERE 
    transactions_id IS NULL OR
    sale_date IS NULL OR
    sale_time IS NULL OR
    gender IS NULL OR
    category IS NULL OR
    quantity IS NULL OR
    cogs IS NULL OR 
    total_sale IS NULL;
```

### 3. Análisis exploratorio básico

- Conteo total de ventas, clientes únicos y categorías distintas.

```sql
SELECT COUNT(*) AS total_sales FROM retail_sales;
SELECT COUNT(DISTINCT customer_id) AS total_customers FROM retail_sales;
SELECT COUNT(DISTINCT category) AS total_categories FROM retail_sales;
```

### 4. Consultas para análisis de negocio

1. **Ventas realizadas el 5 de noviembre de 2022:**

```sql
SELECT * 
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

2. **Transacciones de 'Clothing' con cantidad mayor a 4 en noviembre 2022:**

```sql
SELECT * 
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND DATE_FORMAT(sale_date, '%Y-%m') = '2022-11'
    AND quantity > 4;
```

3. **Total de ventas y número de pedidos por categoría:**

```sql
SELECT 
    category, 
    SUM(total_sale) AS net_sale,
    COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category;
```

4. **Edad promedio de clientes que compraron productos 'Beauty':**

```sql
SELECT 
    ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
WHERE category = 'Beauty';
```

5. **Transacciones con total de venta mayor a 1000:**

```sql
SELECT * 
FROM retail_sales
WHERE total_sale > 1000;
```

6. **Total de transacciones por género y categoría:**

```sql
SELECT 
    category, 
    gender, 
    COUNT(transactions_id) AS total_trans
FROM retail_sales
GROUP BY category, gender
ORDER BY category;
```

7. **Promedio de venta por mes y mejor mes de cada año:**

```sql
WITH monthly_sales AS (
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale) AS avg_sale
    FROM retail_sales
    GROUP BY year, month
)

SELECT 
    year,
    month,
    avg_sale
FROM (
    SELECT 
        year,
        month,
        avg_sale,
        RANK() OVER (
            PARTITION BY year
            ORDER BY avg_sale DESC
        ) AS `rank`
    FROM monthly_sales
) AS ranked
WHERE `rank` = 1;
```

8. **Top 5 clientes con mayores ventas totales:**

```sql
SELECT 
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```

9. **Número de clientes únicos por categoría:**

```sql
SELECT 
    category,
    COUNT(DISTINCT customer_id) AS cnt_unique_customers
FROM retail_sales
GROUP BY category;
```

10. **Conteo de órdenes por turno del día (mañana, tarde, noche):**

```sql
WITH hourly_sale AS (
    SELECT *,
        CASE
            WHEN HOUR(sale_time) < 12 THEN 'Morning'
            WHEN HOUR(sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) AS total_orders
FROM hourly_sale
GROUP BY shift;
```

## Resultados y Conclusiones

- Se identificaron patrones de venta por fecha, categoría y cliente.
- Se detectaron clientes con altos volúmenes de compra.
- Se observaron variaciones en las ventas por turno del día y por mes.
- La limpieza de datos aseguró un análisis confiable sin registros corruptos o incompletos.

## Uso

1. Ejecuta el script para crear la base de datos y la tabla.
2. Carga tus datos en la tabla `retail_sales`.
3. Ejecuta las consultas de análisis para obtener insights.
4. Modifica y extiende las consultas para responder preguntas adicionales de negocio.

