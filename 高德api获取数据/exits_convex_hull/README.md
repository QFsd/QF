# 成都 69 站地铁出入口覆盖多边形

采集日期：2026-10-21。原始出入口来自高德 POI 2.0 的“地铁站出入口”类别（`150501`），对每个参考站点分别做 700 米周边检索和成都市内站名检索。按站名筛选、同坐标去重后，共保留 472 个出入口点。检索响应保存在 `cache/` 和 `text_cache/`，便于复查。

## 结果

| 文件 | 内容 |
| --- | --- |
| `station_exit_convex_hulls_wgs84.geojson` | 69 个多边形，其中 67 个出口点凸包、2 个两出口估算矩形；属性含面积（平方米、平方公里）、周长和出口数。文件名沿用上一版以方便继续引用 |
| `coverage_summary.csv` | 69 站的汇总；`method`、`status`、`length_m`、`width_m`、`width_basis` 区分估算矩形 |
| `station_exit_points_wgs84.geojson` | 472 个出口点，GIS 可直接加载 |
| `station_exit_points.csv` | 出口名称、POI ID、原始 GCJ-02 坐标及转换后的 WGS84 坐标 |
| `two_exit_centerlines_wgs84.geojson` | 宽窄巷子、白果林两个站的出口连线，便于检查矩形长度与方向 |
| `maps/110_amap_static_z16.png` | 宽窄巷子站的高德静态地图图片 |
| `maps/207_amap_static_z16.png` | 白果林站的高德静态地图图片 |

宽窄巷子站检索到 B、D 两个出口点，白果林站检索到 C、D 两个出口点。按用户提出的补充方法，以两点间投影平面直线距离为矩形长度，以 **50 米假设值**为宽度；两出口分别位于矩形两条短边的中点，长边平行于两出口连线。结果如下：

| 站点 | 长度 | 估算宽度 | 面积 | 状态 |
| --- | ---: | ---: | ---: | --- |
| 宽窄巷子 | 163.39 米 | 50 米 | 8,169.30 平方米 | `estimated_rectangle` |
| 白果林 | 134.95 米 | 50 米 | 6,747.25 平方米 | `estimated_rectangle` |

**50 米不是实测站体宽度**，只是本次建模假设。[白果林站的百科条目](https://zh.wikipedia.org/wiki/%E7%99%BD%E6%9E%9C%E6%9E%97%E7%AB%99)有标准段宽 18.7 米的二手记载，但未取得两站一致可核实的实测宽度，因此本版统一按用户允许的 50 米计算。获得可靠站体图纸后，可以替换脚本中的 `TWO_EXIT_WIDTH_M` 并重建。两张高德静态地图图片仍保留，供后续人工处理。

## 计算方法与解释

每站将去重后的出入口坐标由高德 GCJ-02 转换为 WGS84，再投影到 WGS84 / UTM zone 48N（EPSG:32648）计算面积和周长。67 站取出口点集的凸包；两出口站取上述定宽矩形。输出 GeoJSON 的坐标系为 WGS84（EPSG:4326）。同一站名的别称如“成都西站地铁站”已纳入；中医大省医院的重复同坐标记录已合并。周边查询中的邻站出口不会纳入本站。

这里的“覆盖面”是**出口点几何范围的模型**：67 站为出口点凸包，另外两站为定宽矩形。它不包含出口以外的步行可达范围，也不考虑道路通行、障碍、服务半径或客流。高德 POI 搜索文档明确说明搜索不保证返回全量数据，因此缺漏或后续更新的出口可能改变结果；正式用于规划时应结合实地或权威站点资料核实。

API 文档：[高德 POI 2.0](https://lbs.amap.com/api/webservice/guide/api-advanced/newpoisearch)、[高德静态地图](https://lbs.amap.com/api/webservice/guide/api/staticmaps/)。

重建命令（无网络请求）：`python code/build_amap_69_exit_hulls.py`。地图图片下载脚本为 `code/download_two_station_maps.py`，需本机环境变量 `AMAP_KEY` 或 `AMAP_WEB_KEY`。
