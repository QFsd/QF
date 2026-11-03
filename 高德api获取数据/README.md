# 69站高德查询与 OSM 道路对照数据

本目录与 `data/station_buffers_epsg32648/station_master/` 完全独立。原 69 站总表不修改。

## 文件

| 路径 | 内容 | 来源 |
| --- | --- | --- |
| `stations_69_reference.csv` | 69 站名称、WGS84/GCJ-02 坐标与既有线路字段的输入快照 | 既有总表的只读摘录；不是本次 API 查询结果 |
| `metro/metro_station_first_last_by_direction.csv` | 站点、线路、交路方向的首末班时间及查询状态 | 高德公交路径规划中的地铁段；`station_first_train` / `station_last_train` 为上车站时刻，`route_*` 为线路起点时刻，不可混用 |
| `metro/metro_station_coverage_69.csv` | 每站一行的时刻查询覆盖审计 | 根据上表汇总；`has_station_times` 只表示至少有一条方向记录成功，不表示全部方向完整 |
| `poi/amap_poi_broad_1km_by_station.csv` | 20 大类合并搜索的初始覆盖，按站点与 POI ID 去重 | 高德 POI 2.0；最多每站 200 条，不保证全量 |
| `poi/amap_poi_broad_query_status.csv` | 广覆盖查询的完成、未完成与封顶状态 | 同上；`incomplete_pages` 不能当成完整数量 |
| `poi/amap_poi_points_1km_by_station.csv` | 按大类分别查询、按站点与高德 POI ID 去重的补充点数据 | 高德 POI 2.0；搜索结果不保证全量 |
| `poi/amap_poi_query_status.csv` | 每站、每大类的查询完成状态与 200 条上限标记 | 同上；`not_queried` 不能理解为无 POI |
| `poi/amap_poi_observed_union_by_station.csv` | 广覆盖与分类补查按站点和 POI ID 去重后的已观察点合集 | 只代表本次 API 返回集；`found_in_broad`、`found_in_category_search` 标明来源 |
| `roads/osm_walkable_road_segments_1km_by_station.geojson` | 69 站 1 km 圆内裁剪的道路中心线；GeoJSON 几何为 WGS84 | **OSM 四川 2025-01 SHP，不是高德路网** |
| `roads/osm_walkable_road_class_lengths_69stations.csv` | 九类道路在四个圆及四个圆环内的中心线长度，km | 同上；长度在 EPSG:32648 中计算 |
| `roads/road_network_preview_69_and_station20.png` | 69 站覆盖总览与中医大省医院站的道路类别、四个半径示意图 | 由以上两份道路文件绘制；不是成都市完整路网 |
| `roads/amap_traffic/amap_traffic_roads_1km_by_station.geojson` | 高德交通态势接口返回的道路折线，裁剪到各站 1 km 圆；几何为近似转换的 WGS84 | **高德路况道路片段，不是完整路网或步行路网** |
| `roads/amap_traffic/amap_traffic_query_status_69.csv` | 各站原始道路条数、有效折线条数、圆内裁剪条数和查询状态 | 同上；`not_queried` 表示尚未请求 |
| `roads/amap_traffic/amap_traffic_roads_preview_69_station20.png` | 高德交通道路的 69 站总览及中医大省医院站放大图 | 按查询时刻返回的路况着色；不代表步行网络 |

`metro/line_cache/`、`poi/broad_cache/`、`poi/category_cache/` 和 `roads/amap_traffic/cache/` 仅用于断点续跑；不要与最终分析表混淆。

## 本次进度（2026-10-20）

- 地铁：69/69 站至少有一条本站首末班时刻；272 条站点-线路方向记录中，222 条为 `ok`，其余方向须看状态列。
- POI 广覆盖：69/69 站已查，13,311 条站点-POI 记录；68 站触及每站 200 条上限，不可视作 1 km 内完整计数。
- POI 分类补查：科教文化服务（`140000`）69/69 站已查，其中 30 站触及 200 条上限；医疗保健服务（`090000`）只查完 8/69 站，另 61 站为 `not_queried`。其他大类尚未逐类补查。
- 广覆盖与分类补查合并后，按站点 ID + POI ID 去重有 25,230 行；跨站按 POI ID 去重是 21,925 个独立 ID。这两个数均是**已观察返回集**，不是全市或 69 个 1 km 圆的全量 POI 数。
- 道路：按九类 OSM 道路输出 11,791 条站点裁剪线、4,968 行站点-区域-道路类别长度。相邻站点重叠区域的道路会分别记录。
- 高德交通道路：69/69 站查询成功，接口原始返回 4,283 条道路记录，全部带有效 `polyline`；其中 3,418 条折线与本站 1 km 圆相交并写入独立 GeoJSON。这是查询时刻的路况覆盖，不是完整路网。

## 道路口径

`footway`、`living_street`、`pedestrian`、`residential`、`path`、`steps`、`service`、`unclassified`、`unknown`。这是一套**广义可能可步行道路中心线代理**，不是实测人行道长度；普通道路可能不允许步行或没有人行道，`unknown` 在本次 69 站 1 km 区域内实际为 0。`primary`、`secondary`、`tertiary` 不在这套口径中。道路几何按每站分别裁剪，重叠站点会重复出现同一原始道路，因此不能把 GeoJSON 的总长度直接当作全市唯一道路长度。

道路原始数据来自 OpenStreetMap，© OpenStreetMap contributors；使用和再分发时遵循 ODbL：https://www.openstreetmap.org/copyright。

### 高德交通道路的不同口径

高德 `traffic/status/circle` 在每站 GCJ-02 坐标周围查询半径 1,000 m，参数 `level=6`、`extensions=all`。`level` 是**请求时包含的道路等级上限**，不是逐条道路的等级字段；返回列表仅有名称、路况、车行方向、速度、`lcodes` 和 `polyline` 等。输出将 GCJ-02 折线用本项目的近似反算方法转换到 WGS84，再用 EPSG:32648 精确裁剪到每站 1 km 圆；原始响应保存在 `cache/`，可检查未裁剪几何和请求时刻。相邻站点的相同道路会重复返回；同站内也可能有不同方向或路况片段，因此不能直接把全部折线长度相加当作唯一道路总长。

这套接口属于高德的**交通态势高级服务**，现有 Key 的单站测试可用；数据反映查询时刻的路况覆盖，可能随时间变化。它不保证列出所有城市道路，尤其不能代表小区内部道路、独立人行道或步行可达性。论文中的行人网络指标仍应使用明确标注口径的 OSM 数据或另行核验。

## 高德查询运行

先在环境变量中配置 `AMAP_KEY` 或 `AMAP_WEB_KEY`，不要把 Key 写入脚本或 CSV。脚本每次默认最多调用 100 次，触及预算后可以重跑，已完成结果会跳过。

```powershell
X:\Miniconda3\python.exe code\fetch_amap_station_data.py metro --max-requests 100
X:\Miniconda3\python.exe code\fetch_amap_station_data.py poi-broad --max-requests 100
X:\Miniconda3\python.exe code\fetch_amap_station_data.py poi --types 140000,090000,130000,200000 --max-requests 100
X:\Miniconda3\python.exe code\fetch_amap_station_data.py traffic-roads --max-requests 100
X:\Miniconda3\python.exe code\fetch_amap_station_data.py refresh
```

地铁查询按现有站点线路匹配高德线路的各方向交路，再以该站和下游站发起地铁路径规划。只有返回的地铁段确实从目标站上车、且线路方向名称一致时，才采信 `station_start_time`、`station_end_time`。7 号线按“内环”“外环”分别记录；不能只看首末站名，因为环线的首末站可能相同。CSV 同时保留原始 `HHMM` 值和便于阅读的 `*_hhmm` 文本列，例如 `0612` 与 `06:12`。空值和 `station_times_not_returned` 需人工核对运营方时刻表；线路起点时刻不可代填站点时刻。接口返回值未标明工作日/节假日时刻版本，因此不能推断每天相同。高德接口没有提供每班发车间隔的逐站完整时刻表。

POI 先将 20 个大类合并查询，按“页次优先”轮询全部 69 站，让各站先获得初始覆盖；每站最多 200 条。随后可按 20 个大类分别查询，每个类别同样最多取得接口允许的 200 条；超过上限则标记 `query_capped`。广覆盖接口按距离排序，密集站点即使查询半径为 1 km，也可能在几百米内达到 200 条上限；`max_returned_distance_m` 记录实际返回点的最远距离。**任何 `query_capped` 站点的 POI 数量不能作为完整的 1 km 圆或外圈计数。**两份 CSV 是不同阶段的返回集，不能直接相加；如需合并，须按“站点 ID + POI ID”去重。这只是高德搜索返回集，不代表高德内部全量 POI。同一 POI 落入不同站点 1 km 圆时按两个站点各记录一次。高德 GCJ-02 坐标不可直接与 WGS84 或 EPSG:32648 坐标混算。公开发表或分享原始高德数据前，还需核对高德开放平台的现行使用条款。

POI 三份点表都保留高德原始 `lng_gcj02`、`lat_gcj02`，另用本项目已有的迭代反算方法生成 `lng_wgs84`、`lat_wgs84`，再投影出 `x_utm48_m`、`y_utm48_m`（EPSG:32648）。这些是便于空间叠加的**近似转换坐标**，不是高德直接提供的 WGS84 坐标；高精度位置核验仍需独立底图或实测数据。道路 GeoJSON 按标准 WGS84 经纬度存储，其长度在 EPSG:32648 中计算。

状态 `ok` 仅表示当前查询未触及分页上限，并不证明高德内部该范围内不存在其他 POI；`incomplete_pages` 表示本轮尚未取完可用分页，`not_queried` 表示尚未请求。

示例 `--types` 中，`140000` 为科教文化服务，`090000` 为医疗保健服务，`130000` 为政府机构及社会团体，`200000` 为公共设施。这些是高德大类，不应直接等同于论文中定义的“公共服务设施”集合；后续应依据细分 `typecode` 再筛选。

## 来源文档

- 高德公交信息查询：https://lbs.amap.com/api/webservice/guide/api-advanced/bus-inquiry
- 高德路径规划：https://lbs.amap.com/api/webservice/guide/api/direction
- 高德 POI 2.0：https://lbs.amap.com/api/webservice/guide/api-advanced/newpoisearch
- 高德交通态势查询：https://lbs.amap.com/api/webservice/guide/api-advanced/traffic-situation-inquiry
- OSM `sidewalk=*`：https://wiki.openstreetmap.org/wiki/Key:sidewalk
