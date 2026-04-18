<p align="center">
 <img width="100px" src="logo.svg" align="center" alt="Logo" />
 <h2 align="center">UCDT Processing Core</h2>
 <p align="center">数字孪生城市模型处理核心</p>
</p>
<p align="center"> <a href="https://github.com/buildingdata/ucdt-processing-core"><img alt="Github" src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" /></a> <img alt="Debian" src="https://img.shields.io/badge/debian-D80551?style=for-the-badge&logo=debian&logoColor=white"> <img alt="Docker" src="https://img.shields.io/badge/Docker-1D63ED?style=for-the-badge&logo=docker&logoColor=white"> </p>


### 说明

> [!CAUTION]
> 本仓库代码权限为 Private，请勿开源。

这里是 GitHub 展示页入口文档。  

你可以按自己的展示需求手动编辑本文件内容。

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

## 技术栈

- **Python 3.12**
- **依赖管理**: `uv`
- **CLI**: `typer`
- **核心库**: `geopandas`, `shapely`, `pyproj`, `rasterio`, `rtree`, `numpy`, `scipy`
- **数据模型**: `pydantic`
- **配置**: `pyyaml`
- **Excel 导出**: `openpyxl`
- **空间唯一 ID**: `pygeohash`

## 项目结构

```
ucdt-processing-core/
├── cli/                  # CLI 入口
├── gui/                  # 图形界面 (CustomTkinter)
├── core/
│   ├── grid/             # 网格生成
│   ├── dem/              # 高程提取
│   ├── euluc/            # 土地利用分类
│   ├── admin/            # 行政区划富化
│   ├── features/         # 特征工程
│   ├── export/           # 导出器
├── pipelines/            # 管线编排
├── config/               # 配置加载与验证
├── utils/                # 工具函数
├── Dockerfile            # Docker 镜像定义
├── docker-compose.yml    # Docker Compose 编排
└── source-data/          # 输入数据目录
```

## 快速开始

### 1. 安装依赖

```bash
uv sync
```

### 2. 运行处理管线

```bash
# Step 1: 数据聚合（原始数据 → Enriched GPKG）
ucdt preprocess \
  --tile e105_n35_e110_n30 \
  --dem-method centroid

# Step 2: 网格导出（Enriched GPKG → GeoJSON / GPKG / Excel）
ucdt export \
  --gpkg source-data/Enriched/e105_n35_e110_n30_UTM49N.gpkg \
  --origin "108.94000,34.26000" \
  --grid-size 1000 \
  --unit meter \
  --extend "1,1,1,1" \
  --formats "geojson,xlsx" \
  --export-mode split \
  --export-crs wgs84 \
  --floor-rounding floor \
  --unclassified-mode ignore \
  --min-building-area 50 \
  --output ./output

# CSV 批量模式
ucdt export \
  --gpkg source-data/Enriched/e105_n35_e110_n30_UTM49N.gpkg \
  --csv-points grid_points.csv \
  --csv-coord-mode metric \
  --output ./output
```

### 3. 启动图形界面

```bash
ucdt-gui
```

GUI 的数据聚合页支持两种 DEM 模式：

- `centroid`：快速高程聚合（中心点采样，默认）
- `mean`：真实 footprint 均值

GUI 的数据聚合页也支持批量模式：

- 导入 `source-data/example-city-center.csv` 风格的任务表
- 必需列：`经度`、`纬度`、`瓦片`
- 可忽略列：`城市`、`UTM`
- 每一行对应 1 个数据聚合任务，按顺序执行
- 若多行落在同一个 `tile + UTM` 输出目标上，后续重复行会自动跳过

数据聚合会严格要求当前瓦片同时存在 `ODbLPolygon`、`Polygon` 以及 `ChinaAdminDivisonSHP/District/district.shp`，并自动去除与 ODbLPolygon 相交的 Polygon 建筑。

预处理输出的 Enriched GPKG 会自动包含 `administrative_country_*`、`administrative_province_*`、`administrative_city_*`、`administrative_district_*` 字段，后续导出阶段会直接透传这些属性。

该选项会随 GUI 设置一起持久化保存。

### 4. Docker 部署（服务器端 CLI）

```bash
# 构建镜像
docker-compose build

# 运行 CLI 命令（预处理会将 Enriched GPKG 写回挂载的 source-data/Enriched）
docker-compose run --rm ucdt uv run ucdt preprocess --tile e105_n35_e110_n30 --dem-method centroid
```

### 5. 查看版本

```bash
ucdt version
```

## 输出结构

```
output/
├── e105_n35_e110_n30_geojson_YYYYMMDD_HHMMSS/
│   ├── e105_n35_e110_n30_0_0_1.geojson
│   ├── e105_n35_e110_n30_1N_0_1.geojson
│   └── ...
├── e105_n35_e110_n30_gpkg_YYYYMMDD_HHMMSS/
│   ├── e105_n35_e110_n30_0_0_1.gpkg
│   └── ...
└── e105_n35_e110_n30_excel_YYYYMMDD_HHMMSS/
    ├── e105_n35_e110_n30_0_0_1.xlsx
    └── ...
```

## 详细文档

请参阅 [`GUIDE.md`](GUIDE.md) 获取完整的使用指南。
