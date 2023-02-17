---
title: sql语句构造geojson
date: 2023-02-17 17:07:00
categories:
- postgresql
tags:
- geojson
- sql
---

### 背景

1. 数据库存在两张表，一张包含对象的基本属性信息（简称A表），一张包含对象的空间结构信息（简称B表）

2. 数据库仅存在一张表包含对象的属性信息和空间信息。

### 构造`Geojson`

以情况1为例，提取数据库中需要保存的属性信息和空间信息，通过`with as`查询连接两张表，使用`json`函数将结果转换为`json`。

```sql
-- 从表A中提取需要存储的数据信息构建
WITH A AS (select column_name as xxx , ... , column_name as xxx ) from table_name
-- 从表B中提取空间属性信息构造
WITH B as(SELECT column_name as xxx , ... , column_name as xxx  from "parameter" )
-- 连接表A和表B，natural链接自动省略重复字段
WITH C as(SELECT * from A NATURAL inner join B)
-- 至此已完全构造完geojson所需的数据属性

-- 查看提取出来的表结构信息
WITH A AS (select column_name as xxx , ... , column_name as xxx ) from table_name,
	 B as(SELECT column_name as xxx , ... , column_name as xxx  from "parameter" ),
	 C as(SELECT * from A NATURAL inner join B)
	 select * from C
```

已构造`Featurecollection`文件为例，包含两个特定字段`type:"Featurecollection"`、`feature:{}`，`feature`中包含`type`

```json
{
    "type":"FeatureCollection",
    "features":[
        {
            "type":"Feature",
            "properties":{
                "area": 3865207830,
                "text": null
            },
            "id":"polygon.1",
            "geometry":{
                "type":"Polygon",
                "coordinates":[
                    [
                        [
                            116.19827270507814,
                            39.78321267821705
                        ],
                        [
                            116.04446411132814,
                            39.232253141714914
                        ],
                        [
                            116.89590454101562,
                            39.3831409542565
                        ],
                        [
                            116.86981201171876,
                            39.918162846609455
                        ],
                        [
                            116.19827270507814,
                            39.78321267821705
                        ]
                    ]
                ]
            }
        }
    ]
}
```



```sql
-- 从表C中构造feature
WITH feature as(select 'Feature' as type, geometrty,xxx as properties) from C
-- 从feature中构造最后的featurecollection文件
WITH features as(select 'FeatureCollection' as type, array_to_json(array_agg(feature.*)) as features from feature)
     SELECT row_to_json(features.*) from features
```

3. 完整构造`Geojson`

```sql
with A as(SELECT ecgi, azimuth,antenna_hanging_high, st_asgeojson(geometry)::json  as  geometry from parameter_geometry)
     B as(SELECT cell_name,ecgi,azimuth,antenna_hanging_high from "parameter" ),
     C as(SELECT * from A NATURAL inner join B ),
     feature as(SELECT 'Feature' as type, geometry,(SELECT json_strip_nulls(row_to_json(fields)) from (select cell_name,azimuth,antenna_hanging_high) as fields)  as properties from C),
     features as(select 'FeatureCollection' as type, array_to_json(array_agg(feature.*)) as features from feature)
     SELECT row_to_json(features.*) from features
```