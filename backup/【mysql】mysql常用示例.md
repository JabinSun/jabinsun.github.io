```sql
-- 得到正在执行的processlist_id
SELECT
	id 
FROM
	information_schema.PROCESSLIST;

-- 得到与processlist_id对应的thread_id
SELECT
	thread id 
FROM
	PERFORMANCE_SCHEMA.threads 
WHERE
	processlist_id IN (上一步拿到的 processlist_id列表 );

-- 得到正在执行的sql语句
SELECT
	thread_id,
	sql_text 
FROM
	PERFORMANCE_SCHEMA.events_statements_current 
WHERE
	thread_id IN (上一步拿到的 thread_id表 );

-- 得到完整sql(查询哪条sql一直在执行)
SELECT p.id AS processlist_id, t.thread_id, e.sql_text
FROM information_schema.processlist AS p
JOIN performance_schema.threads AS t ON t.processlist_id = p.id
JOIN performance_schema.events_statements_current AS e ON e.thread_id = t.thread_id
WHERE p.id IN (SELECT id FROM information_schema.processlist);
```

```sql
--查询表是否存在
SELECT
	table_name,
	table_schema,
	create_time
FROM
	information_schema.TABLES
WHERE
	table_name = 'TABLE_NAME';
```

```sql
-- 查询表创建时间
SELECT
	table_schema,
	table_name,
	create_time 
FROM
	information_schema.TABLES 
WHERE
	table_name = 'TABLE_NAME';
```

```sql
-- 查询表是否存在某个字段
select
	TABLE_SCHEMA as 'DB_NAME',
	TABLE_NAME as 'TABLE_NAME',
	COLUMN_NAME as 'COLUMN_NAME'
from
	information_schema.COLUMNS
where
	TABLE_SCHEMA in ('db1', 'db2')
	and TABLE_NAME = 'tbl'
	and COLUMN_NAME in ('c1', 'c2');
```

```sql
-- 查询表是否存在某个索引
show index from DB_NAME.TABLE_NAME where Key_name = 'INDEX_NAME';
```