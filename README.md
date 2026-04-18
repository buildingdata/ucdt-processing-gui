<p align="center">
 <img width="100px" src="logo.svg" align="center" alt="Logo" />
 <h2 align="center">UCDT Processing Core</h2>
 <p align="center">数字孪生城市模型处理核心</p>
</p>
<p align="center"> <img alt="Docker" src="https://img.shields.io/github/release/buildingdata/ucdt-processing-gui"> </p>

## 项目简介

UCDT Processing Core 是一个高性能的地理空间数据处理引擎，专为城市尺度建筑数据集设计，能够将原始建筑数据转换为结构化、网格化、可分析的输出，并附带丰富的属性信息（高程、土地利用、形态学特征）。

**适用场景：**
- 城市尺度数据处理
- 能源模拟预处理
- 数字孪生管线
- 大规模批量执行（服务器端）

## 功能说明

- ✅ **网格化处理**：支持米制/度制网格，自动 UTM 投影
- ✅ **高程提取**：基于 DEM 栅格数据的建筑高程计算
- ✅ **土地利用分类**：EULUC 数据匹配（最大相交/质心/最近邻）
- ✅ **行政区划富化**：基于 `ChinaAdminDivisonSHP/District/district.shp` 自动匹配国家、省、市、区县属性
- ✅ **特征工程**：几何特征、形态特征、空间指标（KDTree）
- ✅ **楼层估算**：基于建筑类型的配置驱动估算
- ✅ **多格式导出**：GeoJSON / GPKG / Excel (.xlsx) 按网格拆分或合并导出
- ✅ **建筑 UID**：`building_uid` 基于 WGS84 质心生成 9 位 geohash，全球唯一且与瓦片/坐标系/网格无关
- ✅ **并发处理**：线程分块处理，适合当前 GeoPandas / Shapely / Rasterio 工作负载
- ✅ **双源建筑聚合**：ODbLPolygon + Polygon 自动合并去重，再统一做 LoD1 / DEM / EULUC / 行政区划 聚合
- ✅ **预处理提速**：双源 GeoJSON 并行加载，行政区优先走代表点快路径，边界/歧义建筑仍回退到相交面积判定
- ✅ **瓦片选择器**：GUI Spinner 控件，▲▼ 按钮 ±5° 步进选择瓦片
- ✅ **双坐标模式**：经纬度 / 米制坐标可切换输入
- ✅ **GUI 批量聚合**：数据聚合页支持导入 `example-city-center.csv` 风格任务表，按行顺序批量执行预处理
- ✅ **CSV 批量导入**：通过 POINT_X, POINT_Y 坐标批量生成网格

## License

[GNU General Public License v3](https://github.com/buildingdata/ucdt-processing-gui/blob/main/LICENSE)
