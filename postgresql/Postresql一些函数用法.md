---
title: postgresql一些函数用法
date: 2023-02-17 16:30:00
categories:
- postgresql
tags:
- 数据库
- 函数
---

### `SQL`语句

#### `WITH AS`

```sql
#WITH 子句有助于将复杂的大型查询分解为更简单的表单，
WITH
   name_for_summary_data AS (
      SELECT Statement)
   SELECT columns
   FROM name_for_summary_data
   WHERE conditions <=> (
      SELECT column
      FROM name_for_summary_data)
   [ORDER BY columns]
```

#### `join`

JOIN 子句用于把来自两个或多个表的行结合起来，基于这些表之间的共同字段。

##### `inner join`

* `natural join` `inner join` `inner using`

```sql
SELECT table1.column1, table2.column2...
FROM table1
INNER JOIN table2
ON table1.common_filed = table2.common_field;
```

查询会把 table1 中的每一行与 table2 中的每一行进行比较，找到所有满足连接谓词的行的匹配对。

内连接

##### `CROSS JOIN`

```sql
SELECT table1.column1, table2.column2...
FROM table1 cross join table2
```

交叉连接（CROSS JOIN）把第一个表的每一行与第二个表的每一行进行匹配。如果两个输入表分别有 x 和 y 行，则结果表有 x*y 行。

#### `LEFT OUTER JOIN`

左外连接，首先执行一个内连接。然后，对于表 T1 中不满足表 T2 中连接条件的每一行，其中 T2 的列中有 null 值也会添加一个连接行。因此，连接的表在 T1 中每一行至少有一行。

```sql
SELECT ... FROM table1 LEFT OUTER JOIN table2 ON conditional_expression ...
```

### `SQL`函数

#### `st_asgeojson`

`st_asgeojson(geometry)`

把空间对象输出为`JSON`字符串，当输出结果为`JSON`字符串是，可以使用类型转换将`JSON`字符串转为`JSON`对象。

```sql
st_asegeojson()::json
st_asegeojson()::jsonb
-- postgresql支持的两种数据类型，json和jsonb
-- json存储快，使用慢；jsonb存储慢，使用快。
-- json是对输入的完整拷贝，使用时再去解析，所以它会保留输入的空格，重复键以及顺序等。jsonb是解析输入后保存的二进制，解析时会删除不必要的空格和重复的键，顺序和输入可能也不相同。
```

#### `st_geoFromtext`

根据字符串表示构造几何空间信息，`st_geoFromtext(WKT,SRID)`。

| WKT  | 包含几何文本表示的字符串，eg:LINESTRING(1 2 3, 4 5 6) |
| ---- | ----------------------------------------------------- |
| SRID | 几何信息的参考坐标系，如果不指定则默认为0             |

#### `ST_Centroid`

获取几何对象的中心点，返回空间信息

```sql
SELECT ST_Centroid('MULTIPOINT ( -1 0, -1 2, -1 3, -1 4, -1 7, 0 1, 0 3, 1 1, 2 0, 6 0, 7 8, 9 8, 10 6 )');
------------------------------------------
010100000062277662277602406227766227760A40
```

```sql
WITH A as (SELECT ST_Centroid('MULTIPOINT ( -1 0, -1 2, -1 3, -1 4, -1 7, 0 1, 0 3, 1 1, 2 0, 6 0, 7 8, 9 8, 10 6 )') as geom )
SELECT st_asgeojson(geom) FROM A
```

```json
{"type":"Point","coordinates":[2.307692308,3.307692308]}
```

####  `CONCAT+ST_X+ST_Y生成X,Y坐标`

```sql
SELECT CONCAT(ST_X('POINT(-122.34900 47.65100)'),' ',ST_Y('POINT(-122.34900 47.65100)')) AS geoxy
```

#### `json_strip_nulls` 

递归地删除对象中的值为 null 的字段，非对象字段的 null 值不处理。

#### `row_to_json`

#### `array_to_json`

#### `array_agg`



### 查看数据库

```sql
#查看数据库 
postgres=# \l
                                   List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-------------+----------+----------+-------------+-------------+-----------------------
 aos         | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 aos_default | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 nb-iot      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 nyc         | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 siteplan    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 spatial     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 spatialgis  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
 test        | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 xps         | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
```

### 指定选择表

```sql
# 选择nyc表
postgres=# \c nyc;
psql (11.15, server 9.6.24)
You are now connected to database "nyc" as user "postgres".
```

### 查看表结构

```sql
nyc=# \d city;
                                    Table "public.city"
 Column |         Type          | Collation | Nullable |              Default              
--------+-----------------------+-----------+----------+-----------------------------------
 gid    | integer               |           | not null | nextval('city_gid_seq'::regclass)
 name   | character varying(15) |           |          | 
 number | integer               |           |          | 
 geom   | geometry(Point)       |           |          | 
Indexes:
    "city_pkey" PRIMARY KEY, btree (gid)
    "city_geom_idx" gist (geom)
```

```sql
#删除数据库
DROP DATABASE nbiot;
#创建数据库
CREATE DATABASE nbiot;
\c nbiot;
psql -d nbiot -U postgres -f /home/weihu/public.sql
```

