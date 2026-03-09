Análisis de ejecución
Resultado ETL
Transform Name	Copy	Input	Read	Written	Output	Updated	Rejected	Errors	Buffers Input	Buffers Output	Duration	Speed	Status
src_sales_odoo	0	 33	 0	 33	 0	 0	 0	 0	 0	 0	 0.001"	 478	Finished
trim_fields	0	 0	 33	 33	 0	 0	 0	 0	 0	 0	 0.006"	 524	Finished
replace_null	0	 0	 33	 33	 0	 0	 0	 0	 0	 0	 0.009"	 516	Finished
Select values	0	 0	 33	 33	 0	 0	 0	 0	 0	 0	 0.018"	 458	Finished
dst_staging_sales_clean	0	 0	 33	 33	 33	 0	 0	 0	 0	 0	 0.041"	 344	Finished
Conculta PostgreSQL
docker exec -it postgres.db psql -U user -d odoo
contendor: postgres.db
owner: user
database: odoo

-- Filas cargadas en destino
SELECT COUNT(*) AS total_filas FROM staging.sales_clean;

total_filas 
-------------
          33
(1 row)

-- Distribución por estado
SELECT order_state, COUNT(*) AS total
FROM staging.sales_clean
GROUP BY order_state;

order_state | total 
-------------+-------
 sale        |    33
(1 row)

-- Verificar nulos residuales
SELECT 
    COUNT(*) FILTER (WHERE partner_email IS NULL) AS nulos_email,
    COUNT(*) FILTER (WHERE product_code IS NULL) AS nulos_codigo,
    COUNT(*) FILTER (WHERE qty_ordered IS NULL)  AS nulos_qty,
    COUNT(*) FILTER (WHERE order_date IS NULL)   AS nulos_fecha
FROM staging.sales_clean;

 nulos_email | nulos_codigo | nulos_qty | nulos_fecha 
-------------+--------------+-----------+-------------
           0 |            0 |         0 |           0
(1 row)

-- Muestra de datos
SELECT * FROM staging.sales_clean LIMIT 5;

id | sale_order_id | order_name |   partner_name   |       partner_email       |                 product_name                 | product_code | qty_ordered | unit_price | price_subtotal | order_date | order_state | currency_name |       etl_load_date        |   etl_source
----+---------------+------------+------------------+---------------------------+----------------------------------------------+--------------+-------------+------------+----------------+------------+-------------+---------------+----------------------------+-----------------
 34 |            18 | S00018     | Gemini Furniture | john.b@tech.info          | {\r           