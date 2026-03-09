Identificación flujo
Tabla de origen:
    sale_order, sale_order_line, res_partner, product_product, product_template, res_currency
Tabla de destino: sales_clean
Tipo de carga: Completa
Transformación aplicada:
T1 (string operation)
Elimina espacios delante y atras(Trim)
Campos:
T2 (replace null)
Sustituye campos Null por valores por defecto
Evita que valores faltantes rompan el análisis
T3 (select values)
Convertimos fecha en formato dd-MM-yyyy
Forzamos la presición númerica a 16,4 en campos monetarios