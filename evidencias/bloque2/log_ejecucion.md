Log ejecución pipeline
2026/03/09 09:45:22 - Hop - Pipeline opened.
2026/03/09 09:45:22 - Hop - Launching pipeline [P01_load_sales]...
2026/03/09 09:45:22 - Hop - Started the pipeline execution.
2026/03/09 09:45:22 - P01_load_sales - Executing this pipeline using the Local Pipeline Engine with run configuration 'local'
2026/03/09 09:45:22 - P01_load_sales - Execution started for pipeline [P01_load_sales]
2026/03/09 09:45:22 - dst_staging_sales_clean.0 - Connected to database [odoo_pg_local] (commit=1000)
2026/03/09 09:45:22 - src_sales_odoo.0 - Finished reading query, closing connection.
2026/03/09 09:45:22 - src_sales_odoo.0 - Finished processing (I=33, O=0, R=0, W=33, U=0, E=0)
2026/03/09 09:45:22 - trim_fields.0 - Finished processing (I=0, O=0, R=33, W=33, U=0, E=0)
2026/03/09 09:45:22 - replace_null.0 - Finished processing (I=0, O=0, R=33, W=33, U=0, E=0)
2026/03/09 09:45:22 - Select values.0 - Finished processing (I=0, O=0, R=33, W=33, U=0, E=0)
2026/03/09 09:45:22 - dst_staging_sales_clean.0 - Finished processing (I=0, O=33, R=33, W=33, U=0, E=0)
2026/03/09 09:45:22 - P01_load_sales - Pipeline duration : 0.19 seconds [  0.190" ]
2026/03/09 09:45:22 - P01_load_sales - Execution finished on a local pipeline engine with run configuration 'local'