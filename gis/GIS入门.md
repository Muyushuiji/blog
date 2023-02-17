---
title: GeoTools操作矢量数据
date: 2023-02-17 16:30:00
categories:
- Gis
tags:
- geoTools
- Shp
---

## GeoTools

* 主要提供`GIS`算法，实现各种数据格式大的读写和显示。
* 构建在`OGC`标准之上，用到的比较重要的开源`GIS`工具包是`JTS`

## GeoTools加载shp文件

### 依赖导入

```xml
  <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-geojson</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-shapefile</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-swing</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-epsg-hsql</artifactId>
            <version>${geotools.version}</version>
        </dependency>
```

### 从本地文件中获取shp数据源

```java
public class Quickstart {

    public static void main(String[] args) throws Exception {
        // 选择一个shp文件
        File file = JFileDataStoreChooser.showOpenFile("shp", null);
        if (file == null) {
            return;
        }

        //加载shp图层数据源
        FileDataStore store = FileDataStoreFinder.getDataStore(file);
        SimpleFeatureSource featureSource = store.getFeatureSource();

        // 创建一个地图容器
        MapContent map = new MapContent();
        map.setTitle("Quickstart");



        //创建一个简单的样式，并将样式和shp数据源加载到一个图层上
        Style style = SLD.createSimpleStyle(featureSource.getSchema());
        Layer layer = new FeatureLayer(featureSource, style);

        //在地图容器添加图层
        map.addLayer(layer);

        // 展示地图
        JMapFrame.showMap(map);
    }
}
```

`Swing`选择打开`shp`文件，`FileDataStoreFinder`获得`file`对象的数据源，从数据源中获得`要素类型`。

`MapContent`创建一个地图容器，将特征要素和样式一同加载到图层`Layer`中，并将`Layer`添加进`map`容器展示。



> shp文件获得 -> 文件要素 +SLD样式  -> 生成图层`Layer`加载进地图中 -> 展示

### 从数据源中获取shp文件

>导入postgis依赖

```xml
<dependency>
            <groupId>org.geotools.jdbc</groupId>
            <artifactId>gt-jdbc-postgis</artifactId>
            <version>${geotools.version}</version>
        </dependency>
```

>postgis连接工具类

```java
public class PostGis {
    /**
     * 通过PostGis获取FeatureSource
     *
     * @param host      ip地址
     * @param port      端口号
     * @param database  需要连接的数据库
     * @param userName  用户名
     * @param password  密码
     * @param tableName 需要连接的表名
     * @return
     */
    public static SimpleFeatureSource connAndGetFeatureSource(
            String host, String port, String database,
            String userName, String password, String tableName) {
        Map<String, Object> params = new HashMap<String, Object>(8);
        //需要连接何种数据库，postgis
        params.put(PostgisNGDataStoreFactory.DBTYPE.key, "postgis");
        //ip地址
        params.put(PostgisNGDataStoreFactory.HOST.key, host);
        //端口号
        params.put(PostgisNGDataStoreFactory.PORT.key, new Integer(port));
        //需要连接的数据库
        params.put(PostgisNGDataStoreFactory.DATABASE.key, database);
        //架构
        params.put(PostgisNGDataStoreFactory.SCHEMA.key, "public");
        //需要连接数据库的名称
        params.put(PostgisNGDataStoreFactory.USER.key, userName);
        //数据库的密码
        params.put(PostgisNGDataStoreFactory.PASSWD.key, password);
        try {
            //获取存储空间
            DataStore pgDataStore = DataStoreFinder.getDataStore(params);
            //根据表名获取source
            return pgDataStore.getFeatureSource(tableName);
        } catch (IOException e) {
            throw new RuntimeException(e.getMessage(), e);
        }
    }
}
```

>测试

```java
public class Test {
    public static void main(String[] args) throws IOException {

        SimpleFeatureSource featureSource = getFeatureSourceByPostGis();

        MapContent map = new MapContent();
        map.setTitle("Quickstart");

        Style style = SLD.createSimpleStyle(featureSource.getSchema());
        Layer layer = new FeatureLayer(featureSource, style);
        map.addLayer(layer);
        JMapFrame.showMap(map);
    }

    private static SimpleFeatureSource getFeatureSourceByPostGis() {
        String ip = "192.168.67.22";
        String port = "5432";
        String database = "nyc";
        String userName = "postgres";
        String password = "12345";
        String tableName = "ne_50m_admin_0_countries";
        return PostGis.connAndGetFeatureSource(
                ip, port, database, userName, password, tableName);
    }
}
```

### 将csv数据转换为shp文件

> 读取csv文件构建shp文件的流程

1. 从csv文件中读取表头字段属性，通过`SimpleFeatureTypeBuilder`创建（更灵活）`shp`文件约束，类似于创建表的结构。
2. 

>准备的csv数据样例

```csv
LAT, LON, CITY, NUMBER
46.066667, 11.116667, Trento, 140
44.9441, -93.0852, St Paul, 125
13.752222, 100.493889, Bangkok, 150
45.420833, -75.69, Ottawa, 200
44.9801, -93.251867, Minneapolis, 350
46.519833, 6.6335, Lausanne, 560
48.428611, -123.365556, Victoria, 721
-33.925278, 18.423889, Cape Town, 550
-33.859972, 151.211111, Sydney, 436
41.383333, 2.183333, Barcelona, 914
39.739167, -104.984722, Denver, 869
52.95, -1.133333, Nottingham, 800
45.52, -122.681944, Portland, 840
37.5667,129.681944,Seoul,473
50.733992,7.099814,Bonn,700,2016
```

>pom依赖

```xml
<dependency>
        <groupId>net.sourceforge.javacsv</groupId>
        <artifactId>javacsv</artifactId>
        <version>2.0</version>
    </dependency>
```

>创建feature要素约束

创建`shp`文件约束有两种类型，1. 通过字符串创建。2. 通过`SimpleFeatureTypeBuilder`创建。

```java
//字符串创建
private static SimpleFeatureType createFeatureTypeByStr() throws SchemaException {
        //the_geom" must be of type Point, MultiPoint, MuiltiLineString, MultiPolygon
        //the_geom" is always first, and used for the geometry attribute name
        return DataUtilities.createType("Location",
                "the_geom:Point:srid=4326,name:String,number:Integer");
    }


// SimpleFeatureTypeBuilder创建
private static SimpleFeatureType createFeatureType(Class<? extends Geometry> geometryClazz) {
        SimpleFeatureTypeBuilder builder = new SimpleFeatureTypeBuilder();
        builder.setName("Location");
        // <- Coordinate reference system
        builder.setCRS(DefaultGeographicCRS.WGS84);

        // add attributes in order
        builder.add("the_geom", geometryClazz);
        // <- 15 chars width for name field
        builder.length(15).add("Name", String.class);
        builder.add("number", Integer.class);
        // build the type
        return builder.buildFeatureType();
    }

```

1. `builder`设置要素`typeName`、和`CRS坐标系`。
2. `add`csv表头对应的字段属性，`the_geom`表示当前shp的空间要素且必须为`the_geom`，本样例为`Point`点要素。`city`字段映射为`name`，`numer`字段映射为`number`。


>读取csv文件

```java
private static List<SimpleFeature> readCsv2Feature(SimpleFeatureType type, File file) throws IOException {
        //A list to collect features as we create them.
        List<SimpleFeature> features = new ArrayList<>();
        SimpleFeatureBuilder featureBuilder = new SimpleFeatureBuilder(type);

        GeometryFactory geometryFactory = JTSFactoryFinder.getGeometryFactory();

        CsvReader  csvReader = new CsvReader(file.getAbsolutePath(), ',', StandardCharsets.UTF_8);
        //获取表头
        csvReader.readHeaders();
        while (csvReader.readRecord()) {
            Double latitude = Double.parseDouble(csvReader.get(0));
            Double longitude = Double.parseDouble(csvReader.get(1));
            String name = csvReader.get(2).trim();
            int number = Integer.parseInt(csvReader.get(3).trim());

            Point point = geometryFactory.createPoint(new Coordinate(longitude, latitude));
            featureBuilder.add(point);
            featureBuilder.add(name);
            featureBuilder.add(number);
            SimpleFeature feature = featureBuilder.buildFeature(null);

            features.add(feature);
        }
        return features;
    }
```

1. 根据`shp`要素类型构建`featureBuilder`，获取`csv`文件中的每一行数据，通过`featureBuilder`将每一行数据根据要素构建，并添加至`List<SimpleFeature>`中。
2. 使用`JTS`创建一个点，`JTSFactoryFinder`获取`GeometryFactory`。`geometryFactory`根据`csv`文件中的经纬度数据创建点。
3. `buildFeature`在每次创建`SimpleFeature`时都会对之前构建的要素`reset()`清零。

>创建`shp`文件

1. 构建`shp`文件对象

```java
private static File getNewShapeFile(File csvFile) {
        String path = csvFile.getAbsolutePath();
        String newPath = path.substring(0, path.length() - 4) + ".shp";

        JFileDataStoreChooser chooser = new JFileDataStoreChooser("shp");
        chooser.setDialogTitle("Save shapefile");
        chooser.setSelectedFile(new File(newPath));

        int returnVal = chooser.showSaveDialog(null);

        if (returnVal != JFileDataStoreChooser.APPROVE_OPTION) {
            // the user cancelled the dialog
            System.exit(0);
        }
        File newFile = chooser.getSelectedFile();
        if (newFile.equals(csvFile)) {
            System.out.println("Error: cannot replace " + csvFile);
            System.exit(0);
        }
        return newFile;
    }
```

* 在`csv`文件同目录下创建`shp`文件路径，`JFileDataStoreChooser`构建选择框标题和保存的文件路径。

* 构建`newFile`，避免`file`类型为`csv`的情况。



2. 创建`shp`文件

```java
public static void main(String[] args) throws Exception {
        UIManager.setLookAndFeel(UIManager.getCrossPlatformLookAndFeelClassName());

        File file = JFileDataStoreChooser.showOpenFile("csv", null);
        if (file == null) {
            return;
        }

        //Create a FeatureType
        SimpleFeatureType type = createFeatureType(Point.class);

        //将csv转换为feature
        List<SimpleFeature> features = readCsv2Feature(type, file);


        //Get an output file name and create the new shapefile
        File newFile = getNewShapeFile(file);

        ShapefileDataStoreFactory dataStoreFactory = new ShapefileDataStoreFactory();

        Map<String, Serializable> params = new HashMap<>(2);
        params.put("url", newFile.toURI().toURL());
        params.put("create spatial index", Boolean.TRUE);

        ShapefileDataStore newDataStore =
                (ShapefileDataStore) dataStoreFactory.createNewDataStore(params);

        //TYPE is used as a template to describe the file contents
        newDataStore.createSchema(type);

        //Write the features to the shapefile
        Transaction transaction = new DefaultTransaction("create");

        String typeName = newDataStore.getTypeNames()[0];
        SimpleFeatureSource featureSource = newDataStore.getFeatureSource(typeName);

        if (featureSource instanceof SimpleFeatureStore) {
            SimpleFeatureStore featureStore = (SimpleFeatureStore) featureSource;
            SimpleFeatureCollection collection = new ListFeatureCollection(type, features);
            featureStore.setTransaction(transaction);
            try {
                featureStore.addFeatures(collection);
                transaction.commit();
            } catch (Exception problem) {
                problem.printStackTrace();
                transaction.rollback();
            } finally {
                transaction.close();
            }
            System.exit(0);
        } else {
            System.out.println(typeName + " does not support read/write access");
            System.exit(1);
        }
```

1. 通过`dataStoreFactory`并结合特定的参数创建`ShapefileDataStore`，设置`shp`文件的uri地址和空间地理索引。
2. `newDataStore`创建表约束，就是表的字段属性。
3. 通过`newDataStore`获取要素信息featureSource`，开启事务。
4. 通过要素集合的类型和要素数据创建集合`collection`，将要素集合`collection`放入featureStore。提交事务，创建`shp`文件。



## 数据类型格式

### Shape

矢量图形格式，使用点、线、多边形存储要素的形状，它能够保存几何图形的位置及相关属性。（不能存储拓扑结构）

一个`shapefile`由若干个文件组成，空间信息和属性信息分离存储。

* `*.shp`:保存元素的几何实体，存储几何要素的空间信息（XY坐标）。
* `*.shx`：保存几何实体的索引，存储的是有关`*.shp`存储的索引信息。
* `*.dbf`：数据库，保存元素的属性信息。

`GeoTools`加载shp文件从磁盘读取并写入磁盘，不通过内存记载。

### GeoJson

以`json`的语法存储地理数据

```json
{
  "type": "FeatureCollection",
  "features": [
        {"type":"Feature",
        "properties":{},
        "geometry":{
            "type":"Point",
            "coordinates":[105.380859375,31.57853542647338]
            }
        }
    ]
}
```

`GeoJson`将所有的地理要素分为`Point`、`MultiPoint`、`LineString`、`MultiLineString`、`Polygon`、`MultiPolygon`、`GeometryCollection`。首先将这些要素封装到单个的`geometry`里，然后作为一个个的`Feature`（要素）放到`FeatureCollection`要素集合中。

### WKT



### 数据类型之间的转换

