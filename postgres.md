
* Cancell the process
```
  SELECT pg_cancel_backend(<pid of the process>)
 ```
* Terminate the process
```
  SELECT pg_terminate_backend(<pid of the process>)
 ``` 
* TOP-20 huge tables
```
SELECT nspname || '.' || relname AS "relation",
    pg_size_pretty(pg_relation_size(C.oid)) AS "size"
  FROM pg_class C
  LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
  WHERE nspname NOT IN ('pg_catalog', 'information_schema')
  ORDER BY pg_relation_size(C.oid) DESC
  LIMIT 20;
```
