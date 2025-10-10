都是地理空间数据格式，用来表示地理要素。

GeoJSON 是基于 JSON 的地理空间数据格式。有以下特点：
- 支持多种几何类型。
- 可以包含属性数据，用于描述地理要素的特征。

示例，在线预览网站 https://geojson.io/ ：
```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [102.0, 0.5]
  },
  "properties": {
    "name": "Example Point"
  }
}
```

WKT 是一种文本格式，有以下特点：
- 简单明了，仅包含几何信息，不包含属性数据。

示例，在线预览网站 https://wktmap.com/ ：
```
POLYGON ((-64.8 32.3, -65.5 18.3, -80.3 25.2, -64.8 32.3))
```

对比可以发现，WKT 只包含 GeoJSON `geometry` 属性部分的信息，两者可以互相转换。
GeoJSON 和 WKT 都支持的几何类型：

| **几何类型** | ​**GeoJSON**         | ​**WKT**             |
| -------- | -------------------- | -------------------- |
| 点        | `Point`              | `POINT`              |
| 线        | `LineString`         | `LINESTRING`         |
| 面        | `Polygon`            | `POLYGON`            |
| 多点       | `MultiPoint`         | `MULTIPOINT`         |
| 多线       | `MultiLineString`    | `MULTILINESTRING`    |
| 多面       | `MultiPolygon`       | `MULTIPOLYGON`       |
| 几何集合     | `GeometryCollection` | `GEOMETRYCOLLECTION` |
