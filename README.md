SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' AND rownum <= 999 AND owner = :1 AND table_name = :2
UNION ALL
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' AND rownum <= 999 AND owner = :3 AND table_name = :4
UNION ALL
...
