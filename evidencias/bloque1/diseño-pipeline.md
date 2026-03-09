# Diseño del Pipeline

## 1. Identificar flujo de datos

### Acceso al contenedor de PostgreSQL

```bash
docker exec -it postgres.db psql -U user -d odoo
```

---

# 2. Exploración de funciones

```sql
\df
```

Resultado (extracto):

```
List of functions

 Schema | Name                    | Result data type | Argument data types                            | Type
--------|-------------------------|------------------|------------------------------------------------|------
 public | gin_extract_query_trgm  | internal         | text, internal, smallint, internal, internal   | func
 public | gin_extract_value_trgm  | internal         | text, internal                                 | func
 public | gin_trgm_consistent     | boolean          | internal, smallint, text, integer, internal    | func
```

---

# 3. Exploración de tablas

### Tablas relevantes

```sql
\dt sale*
\dt res*
\dt product*
```

Resultado:

```
Schema | Name                                              | Type  | Owner
-------|---------------------------------------------------|-------|------
public | account_tax_sale_order_discount_rel               | table | user
public | account_tax_sale_order_line_rel                   | table | user
public | product_document_sale_pdf_form_field_rel          | table | user
public | product_template_attribute_value_sale_order_line_rel | table | user
public | quotation_document_sale_order_rel                 | table | user
public | quotation_document_sale_pdf_form_field_rel        | table | user
public | sale_advance_payment_inv                          | table | user
public | sale_advance_payment_inv_sale_order_rel           | table | user
public | sale_mass_cancel_orders                           | table | user
public | sale_order                                        | table | user
public | sale_order_cancel                                 | table | user
public | sale_order_discount                               | table | user
public | sale_order_line                                   | table | user
```

Tablas principales del flujo:

- `sale_order`
- `sale_order_line`

---

# 4. Estructura de tablas

## Tabla `sale_order`

```sql
\d sale_order
```

Columnas relevantes:

| Column | Type | Description |
|------|------|------|
| id | integer | ID del pedido |
| name | varchar | Número de orden |
| partner_id | integer | Cliente |
| date_order | timestamp | Fecha del pedido |
| state | varchar | Estado |
| amount_untaxed | numeric | Total sin impuestos |
| amount_tax | numeric | Impuestos |
| amount_total | numeric | Total del pedido |

Primary Key:

```
sale_order_pkey (id)
```

---

## Tabla `sale_order_line`

```sql
\d sale_order_line
```

Columnas relevantes:

| Column | Type | Description |
|------|------|------|
| id | integer | ID de línea |
| order_id | integer | Relación con sale_order |
| product_id | integer | Producto |
| name | text | Descripción |
| product_uom_qty | numeric | Cantidad |
| price_unit | numeric | Precio unitario |
| price_subtotal | numeric | Subtotal |

Primary Key:

```
sale_order_line_pkey (id)
```

---

# 5. Datos disponibles

## Ejemplo `sale_order`

```sql
SELECT id, name, partner_id, date_order, amount_total, state
FROM sale_order
LIMIT 10;
```

Resultado:

| id | name | partner_id | date_order | amount_total | state |
|---|---|---|---|---|---|
|1|S00001|10|2026-02-05 13:13:00|1740.00|draft|
|2|S00002|12|2026-02-05 13:13:00|2947.50|draft|
|3|S00003|12|2026-03-05 13:13:43|377.50|draft|
|5|S00005|10|2026-02-05 13:13:00|405.00|draft|
|19|S00019|8|2026-02-05 13:13:00|1740.00|sent|
|6|S00006|16|2026-03-05 13:13:43|750.00|sale|
|8|S00008|11|2026-03-05 13:13:43|462.00|sale|
|4|S00004|11|2026-03-05 13:13:43|2240.00|sale|
|7|S00007|11|2026-03-05 13:13:43|1706.00|sale|
|9|S00009|11|2026-02-26 13:13:43.712546|654.00|sale|

---

## Ejemplo `sale_order_line`

```sql
SELECT
  id,
  order_id,
  product_id,
  name,
  product_uom_qty,
  price_unit,
  price_subtotal
FROM sale_order_line
LIMIT 10;
```

Resultado:

| id | order_id | product_id | name | qty | price_unit | subtotal |
|---|---|---|---|---|---|---|
|1|1|32|Pantallas de bloque acústico (Madera)|3|295|885|
|2|1|6|Lámpara de oficina|5|145|725|
|3|1|5|Silla de oficina|2|65|130|
|4|2|3|Diseño interior virtual|24|75|1800|
|5|2|4|Área de casa virtual|30|38.25|1147.5|
|6|3|3|Diseño interior virtual|10|30.75|307.5|
|7|3|5|Silla de oficina|1|70|70|
|12|5|6|Lámpara de oficina|1|405|405|
|40|19|32|Pantallas de bloque acústico (Madera)|3|295|885|
|41|19|6|Lámpara de oficina|5|145|725|

---

# 6. Próximo paso del pipeline

Crear **schema de staging** y tablas de destino para el proceso ETL.

## Crear schema staging

```sql
CREATE SCHEMA staging;
```

## Crear tabla staging

```sql
CREATE TABLE staging.sale_orders (
    order_id INTEGER,
    order_name TEXT,
    partner_id INTEGER,
    order_date TIMESTAMP,
    total_amount NUMERIC,
    state TEXT
);
```

---

- Verificar creación

\dt staging.*
\d staging.sales_clean

           List of relations
 Schema  |    Name     | Type  | Owner
---------+-------------+-------+-------
 staging | sales_clean | table | user
(1 row)

                                              Table "staging.sales_clean"
     Column     |            Type             | Collation | Nullable |                     Default
----------------+-----------------------------+-----------+----------+-------------------------------------------------
 id             | integer                     |           | not null | nextval('staging.sales_clean_id_seq'::regclass)
 sale_order_id  | integer                     |           |          |
 order_name     | character varying(100)      |           |          |
 partner_name   | character varying(255)      |           |          |
 partner_email  | character varying(255)      |           |          |
 product_name   | character varying(255)      |           |          |
 product_code   | character varying(100)      |           |          |
 qty_ordered    | numeric(16,4)               |           |          | 0
 unit_price     | numeric(16,4)               |           |          | 0
 price_subtotal | numeric(16,4)               |           |          | 0
 order_date     | date                        |           |          |
 order_state    | character varying(50)       |           |          |
 currency_name  | character varying(10)       |           |          |
 etl_load_date  | timestamp without time zone |           |          | now()
 etl_source     | character varying(100)      |           |          | 'hop_pipeline_v1'::character varying
Indexes:
    "sales_clean_pkey" PRIMARY KEY, btree (id)

- Dar permisos

GRANT ALL PRIVILEGES ON SCHEMA staging TO "user";
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA staging TO "user";
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA staging TO "user";












- Verificar

SELECT COUNT(*) FROM staging.sales_clean;






## count
-------
     0
(1 row)

configuración conexión

SELECT
    so.id                        AS sale_order_id,
    so.name                      AS order_name,
    rp.name                      AS partner_name,
    rp.email                     AS partner_email,
    pt.name                      AS product_name,
    pt.default_code              AS product_code,
    sol.product_uom_qty          AS qty_ordered,
    sol.price_unit               AS unit_price,
    sol.price_subtotal           AS price_subtotal,
    so.date_order                AS order_date,
    so.state                     AS order_state,
    rc.name                      AS currency_name
FROM
    sale_order so
    JOIN sale_order_line sol  ON sol.order_id = so.id
    JOIN res_partner rp       ON rp.id = so.partner_id
    JOIN product_product pp   ON pp.id = sol.product_id
    JOIN product_template pt  ON pt.id = pp.product_tmpl_id
    JOIN res_currency rc      ON rc.id = so.currency_id
WHERE
    so.state IN ('sale', 'done')
ORDER BY so.date_order ASC;
```











