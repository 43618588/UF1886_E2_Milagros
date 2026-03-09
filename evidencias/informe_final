# Informe Técnico Final – Proceso ETL Odoo 18 → Staging

**Alumno:**        [Tu nombre]  
**Fecha:**         04/03/2026
**Entorno:**       Docker + PostgreSQL + Odoo 18 (datos demo) + Apache Hop GUI (Windows)  
**Pipeline:**      `P01_load_sales.hpl`  
**Tabla destino:** `staging.sales_clean`

---

## 1. Diseño del Pipeline

### 1.1 Arquitectura general

```
[Odoo 18 – PostgreSQL Docker]
        │
        │  6 tablas unidas por JOIN
        │  sale_order + sale_order_line + res_partner
        │  product_product + product_template + res_currency
        ▼
┌─────────────────┐
│   Table Input   │  SQL con filtro state IN ('sale','done')
└────────┬────────┘
         ▼
┌──────────────────────┐
│   String Operations  │  TRANSFORM 1 – Limpieza de espacios
└────────┬─────────────┘
         ▼
┌──────────────────────┐
│      If Null         │  TRANSFORM 2 – Sustitución de nulos
└────────┬─────────────┘
         ▼
┌──────────────────────┐
│    Select Values     │  TRANSFORM 3 – Conversión de tipos
└────────┬─────────────┘
         ▼
┌──────────────────────┐
│    Table Output      │  → staging.sales_clean (Truncate + Insert)
└──────────────────────┘
```

### 1.2 Configuración de la conexión

| Parámetro       | Valor           |
|-----------------|-----------------|
| Nombre          | odoo_pg_local   |
| Motor           | PostgreSQL      |
| Host            | localhost       |
| Puerto          | 5432            |
| Base de datos   | odoo            |
| Usuario         | user            |

### 1.3 Tabla origen — query SQL

```sql
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
ORDER BY
    so.date_order ASC;
```

### 1.4 Tabla destino — `staging.sales_clean`

| Columna          | Tipo              | Restricción       |
|------------------|-------------------|-------------------|
| id               | SERIAL            | PK                |
| sale_order_id    | INTEGER           | NOT NULL          |
| order_name       | VARCHAR(100)      |                   |
| partner_name     | VARCHAR(255)      |                   |
| partner_email    | VARCHAR(255)      |                   |
| product_name     | VARCHAR(255)      |                   |
| product_code     | VARCHAR(100)      |                   |
| qty_ordered      | NUMERIC(16,4)     | DEFAULT 0         |
| unit_price       | NUMERIC(16,4)     | DEFAULT 0         |
| price_subtotal   | NUMERIC(16,4)     | DEFAULT 0         |
| order_date       | DATE              |                   |
| order_state      | VARCHAR(50)       |                   |
| currency_name    | VARCHAR(10)       |                   |
| etl_load_date    | TIMESTAMP         | DEFAULT NOW()     |
| etl_source       | VARCHAR(100)      | DEFAULT 'hop_pipeline_v1' |

---

## 2. Transformaciones Aplicadas

### T1 — String Operations (Limpieza de espacios)

**Transform Hop:** `String Operations`  
**Nombre en pipeline:** `trim_fields`

| Campo de entrada | Campo de salida | Operación |
|------------------|-----------------|-----------|
| order_name       | order_name      | Trim both |
| partner_name     | partner_name    | Trim both |
| partner_email    | partner_email   | Trim both |
| product_name     | product_name    | Trim both |
| product_code     | product_code    | Trim both |
| order_state      | order_state     | Trim both |
| currency_name    | currency_name   | Trim both |

**Motivación:**  
Los datos ingresados manualmente en Odoo frecuentemente contienen espacios
involuntarios al inicio o al final de los campos de texto. Estos espacios pueden
causar problemas en comparaciones, búsquedas y agrupaciones en la capa de análisis.

---

### T2 — If Null (Sustitución de nulos)

**Transform Hop:** `If Null`  
**Nombre en pipeline:** `replace_nulls`

| Campo          | Valor sustituto               | Justificación                                |
|----------------|-------------------------------|----------------------------------------------|
| partner_email  | <sin_email@desconocido.com>     | Email NULL rompe validaciones posteriores    |
| product_code   | SIN-CODIGO                    | Referencia interna opcional en Odoo          |
| product_name   | Producto sin nombre           | Evita registros ambiguos en análisis         |
| qty_ordered    | 0                             | NULL en cantidad distorsiona agregaciones    |
| unit_price     | 0                             | NULL en precio impide cálculos financieros   |
| price_subtotal | 0                             | Campo calculado; se neutraliza si falta      |
| currency_name  | USD                           | Moneda por defecto del sistema               |

**Motivación:**  
En Odoo 18 con datos demo, campos como `partner_email` y `product_code` son
opcionales y pueden estar vacíos. Los valores NULL en campos numéricos producen
resultados incorrectos en sumas y promedios en herramientas de BI.

---

### T3 — Select Values (Conversión de tipos de datos)

**Transform Hop:** `Select Values` (pestaña Meta-data)  
**Nombre en pipeline:** `convert_types`

| Campo          | Tipo origen    | Tipo destino | Longitud | Precisión | Formato    |
|----------------|----------------|--------------|----------|-----------|------------|
| sale_order_id  | Integer        | Integer      | 10       |           |            |
| order_name     | String         | String       | 100      |           |            |
| partner_name   | String         | String       | 255      |           |            |
| partner_email  | String         | String       | 255      |           |            |
| product_name   | String         | String       | 255      |           |            |
| product_code   | String         | String       | 100      |           |            |
| qty_ordered    | Number         | BigNumber    | 16       | 4         |            |
| unit_price     | Number         | BigNumber    | 16       | 4         |            |
| price_subtotal | Number         | BigNumber    | 16       | 4         |            |
| order_date     | Timestamp      | Date         | 10       |           | yyyy-MM-dd |
| order_state    | String         | String       | 50       |           |            |
| currency_name  | String         | String       | 10       |           |            |

**Motivación:**  
Odoo almacena precios como `float8` (punto flotante) que puede generar imprecisión
en cálculos financieros. Se convierte a `BigNumber` con precisión `16,4`.
El campo `date_order` incluye hora en Odoo — se reduce a fecha simple `DATE`
para homogenizar el análisis temporal.

---

## 3. Ejecución del Pipeline

### 3.1 Datos de la ejecución

| Parámetro              | Valor                              |
|------------------------|------------------------------------|
| Fecha de ejecución     | [COMPLETAR con tu fecha real]      |
| Hora de inicio         | [COMPLETAR]                        |
| Hora de fin            | [COMPLETAR]                        |
| Duración               | [COMPLETAR]                        |
| Herramienta            | Apache Hop GUI – Windows           |
| Run configuration      | Local                              |

### 3.2 Métricas de ejecución

| Métrica                | Valor                              |
|------------------------|------------------------------------|
| Filas leídas (origen)  | [COMPLETAR con datos del log]      |
| Filas escritas (destino)| [COMPLETAR con datos del log]     |
| Filas rechazadas       | [COMPLETAR]                        |
| Errores                | [COMPLETAR]                        |
| Estado final           | [Pipeline finished / failed]       |

### 3.3 Verificación en base de datos

```sql
-- Ejecutado tras la carga:
SELECT COUNT(*) AS total_filas FROM staging.sales_clean;
-- Resultado: [COMPLETAR]

SELECT order_state, COUNT(*) AS total, SUM(price_subtotal) AS revenue
FROM staging.sales_clean
GROUP BY order_state;
-- Resultado: [COMPLETAR]
```

### 3.4 Verificación de nulos residuales

```sql
SELECT
    COUNT(*) FILTER (WHERE partner_email IS NULL) AS nulos_email,
    COUNT(*) FILTER (WHERE product_code IS NULL)  AS nulos_codigo,
    COUNT(*) FILTER (WHERE qty_ordered IS NULL)   AS nulos_qty,
    COUNT(*) FILTER (WHERE order_date IS NULL)    AS nulos_fecha
FROM staging.sales_clean;
-- Resultado esperado: 0 en todos los campos
-- Resultado obtenido: [COMPLETAR]
```

---

## 4. Incidencias Detectadas

### 4.1 Registro de incidencias

| # | Tipo               | Clasificación       | Transform afectado | Detectada en |
|---|--------------------|--------------------|--------------------|--------------|
| 1 | [COMPLETAR]        | [COMPLETAR]        | [COMPLETAR]        | [COMPLETAR]  |

> Clasificación posible: `Gestor de datos` / `ETL` / `Integridad de datos` / `Conexión`

---

### 4.2 Detalle de la incidencia

**Tipo de incidencia:** [COMPLETAR — ej: Clave duplicada]  
**Clasificación:** [COMPLETAR — ej: Gestor de datos]  
**Transform que falla:** [COMPLETAR — ej: Table Output]

**Mensaje de error en log:**

```
[PEGAR AQUÍ EL MENSAJE EXACTO DEL LOG DE HOP]
```

**Análisis de causa raíz:**

[COMPLETAR según el tipo de error — ejemplos:]

- **Clave duplicada:** El pipeline se ejecutó dos veces sin tener activado `Truncate table`, generando conflicto en la clave primaria `id` de `staging.sales_clean`.
- **Columna inexistente:** El SQL del `Table Input` referencia una columna con nombre incorrecto o que no existe en la versión instalada de Odoo 18.
- **Error de conexión:** El contenedor de PostgreSQL estaba detenido o el puerto 5432 no estaba mapeado correctamente en el host Windows.
- **Tipo de dato incorrecto:** El stream de datos en Hop llegó al `Table Output` con un tipo incompatible con el definido en `staging.sales_clean`.

---

## 5. Acciones Correctivas

### 5.1 Acción aplicada para resolver la incidencia

**Incidencia:** [COMPLETAR]  
**Acción aplicada:**

[COMPLETAR según tu caso — ejemplos:]

**Si fue clave duplicada:**

```
1. Doble clic en Table Output
2. Activar ☑ "Truncate table"
3. Guardar pipeline (Ctrl+S)
4. Volver a ejecutar (F9)
```

**Si fue columna inexistente:**

```bash
# Verificar columnas reales en PostgreSQL
docker exec -it db psql -U user -d odoo -c "\d sale_order_line"
# Corregir el nombre en el SQL del Table Input
# Guardar y volver a ejecutar
```

**Si fue error de conexión:**

```bash
# Reiniciar el contenedor
docker start db
# Verificar en Hop: Database Connection > Test
# Volver a ejecutar
```

**Si fue tipo de dato incorrecto:**

```
1. Abrir Select Values → pestaña Meta-data
2. Corregir el tipo del campo afectado
3. Guardar pipeline (Ctrl+S)
4. Volver a ejecutar (F9)
```

### 5.2 Resultado tras la corrección

| Métrica              | Antes (con error) | Después (corregido) |
|----------------------|-------------------|---------------------|
| Estado del pipeline  | Failed            | [COMPLETAR]         |
| Filas escritas       | 0                 | [COMPLETAR]         |
| Errores en log       | [COMPLETAR]       | 0                   |

---

## 6. Verificación Posterior

### 6.1 Verificación en Apache Hop (log)

Líneas clave del log tras la corrección exitosa:

```
[PEGAR AQUÍ LAS LÍNEAS FINALES DEL LOG DESPUÉS DE LA CORRECCIÓN]
```

Indicadores esperados:

```
Pipeline finished    → OK
Rows read:           → coincide con origen
Rows written:        → coincide con filas leídas
Errors:              → 0
```

### 6.2 Verificación en PostgreSQL

```sql
-- Conteo final
SELECT COUNT(*) FROM staging.sales_clean;
-- [COMPLETAR con resultado]

-- Integridad de datos
SELECT
    MIN(order_date)       AS primera_venta,
    MAX(order_date)       AS ultima_venta,
    COUNT(DISTINCT sale_order_id) AS pedidos_unicos,
    SUM(price_subtotal)   AS revenue_total,
    COUNT(*) FILTER (WHERE partner_email = 'sin_email@desconocido.com') AS emails_sustituidos,
    COUNT(*) FILTER (WHERE product_code = 'SIN-CODIGO') AS codigos_sustituidos
FROM staging.sales_clean;
-- [COMPLETAR con resultado]
```

### 6.3 Conclusión de la verificación

| Pregunta                              | Respuesta          |
|---------------------------------------|--------------------|
| ¿Finalizó correctamente?              | [SÍ / NO]          |
| ¿Coinciden filas origen y destino?    | [SÍ / NO – explicar si no] |
| ¿Se detectaron advertencias?          | [SÍ / NO – detallar]|
| ¿Hay nulos residuales?                | [SÍ / NO]          |
| ¿Los tipos de datos son correctos?    | [SÍ / NO]          |

---

## 7. Medidas Preventivas

### 7.1 Para evitar recurrencia de las incidencias detectadas

| Medida                          | Descripción                                                                 | Prioridad |
|---------------------------------|-----------------------------------------------------------------------------|-----------|
| Activar Truncate por defecto    | En Table Output siempre activar `Truncate table` en cargas completas        | Alta      |
| Validar conexión antes de ejecutar | Usar `Database Connection > Test` antes de cada ejecución               | Alta      |
| Documentar versión de Odoo      | Registrar versión exacta del módulo `sale` para detectar cambios de esquema | Media     |
| Preview en cada transform       | Usar clic derecho > Preview rows en cada transform antes de ejecutar todo   | Alta      |
| Verificar tipos en Meta-data    | Revisar pestaña Meta-data de Select Values tras cualquier cambio en Odoo    | Media     |

### 7.2 Buenas prácticas ETL generales

**Control de ejecución:**

```sql
-- Crear tabla de log de ejecuciones (para producción futura)
CREATE TABLE IF NOT EXISTS staging.etl_log (
    id            SERIAL PRIMARY KEY,
    pipeline_name VARCHAR(100),
    start_time    TIMESTAMP,
    end_time      TIMESTAMP,
    rows_read     INTEGER,
    rows_written  INTEGER,
    status        VARCHAR(20),
    error_message TEXT
);
```

**Evolución hacia carga incremental:**

```sql
-- Filtro incremental por fecha de modificación
WHERE so.write_date > '{{LAST_LOAD_DATE}}'
  AND so.state IN ('sale', 'done')
-- Requiere tabla de control: staging.etl_log
```

**Validación de calidad post-carga:**

```sql
-- Ejecutar siempre después de cada carga
SELECT
    CASE
        WHEN COUNT(*) = 0 THEN 'ERROR: tabla vacía'
        WHEN COUNT(*) FILTER (WHERE qty_ordered < 0) > 0 THEN 'WARN: cantidades negativas'
        WHEN COUNT(*) FILTER (WHERE price_subtotal < 0) > 0 THEN 'WARN: precios negativos'
        ELSE 'OK: datos válidos'
    END AS estado_calidad
FROM staging.sales_clean;
```

### 7.3 Checklist pre-ejecución

```
□ Verificar que el contenedor Docker está activo (docker ps)
□ Probar conexión en Hop (Database Connection > Test)  
□ Confirmar que staging.sales_clean existe (\dt staging.*)
□ Revisar que Table Output tiene Truncate activado
□ Hacer Preview en Table Input para confirmar datos origen
□ Ejecutar pipeline completo (F9)
□ Verificar COUNT(*) en staging.sales_clean tras la carga
□ Guardar log en evidencias/BLOQUE2/log_ejecucion.txt
```

---

## 8. Resumen Ejecutivo

| Aspecto                 | Detalle                                              |
|-------------------------|------------------------------------------------------|
| Pipeline                | etl_sales_odoo_to_staging.hpl                        |
| Origen                  | 6 tablas de Odoo 18 (datos demo)                     |
| Destino                 | staging.sales_clean                                  |
| Tipo de carga           | Completa (Truncate + Insert)                         |
| Transforms aplicados    | String Operations, If Null, Select Values            |
| Incidencias detectadas  | [COMPLETAR]                                          |
| Incidencias resueltas   | [COMPLETAR]                                          |
| Estado final            | [COMPLETAR]                                          |
| Filas cargadas          | [COMPLETAR]                                          |
| Calidad de datos        | Sin nulos en campos críticos / tipos correctos       |

---

*Documento generado como evidencia técnica del ejercicio ETL – Apache Hop + Odoo 18*