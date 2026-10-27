# 三国风光荷与日前天气条件 / Joint wind–PV–load and day-ahead weather data

本地整理版本：2026-10-27。覆盖德国（DE）、奥地利（AT）、法国（FR），2021-03-24 至 2025-12-31，共 1,744 天。该包来自已保存的论文数据快照；未重新下载、替换或修改观测值。

## 文件

- `data/hourly_data.csv`：125,568 行，每行一个 UTC 小时、一个国家；可直接用 Excel、Python 或 R 读取。
- `data/wind_pv_load_weather_2021_2025.npz`：保留原研究快照全部 29 个数组，包括最终输入、缺失掩码和来源信息。
- `splits/*.csv`：按原 NPZ 的 split 标签逐日导出；day_index_zero_based 为 NPZ 第一维的零起始索引。
- `DATA_DICTIONARY.json`：数组形状、字段顺序、数据来源代码及数量。
- `VALIDATION.json`：本次对齐与完整性检查结果。
- `SHA256SUMS.txt`：包内文件的 SHA-256 校验值。

## 时间与划分

全部时间为 UTC，不按各国夏令时调整。每个国家包含 41,856 个小时。训练集为 2021-03-24 至 2023-12-31（1,013 天）；验证集为 2024 年（366 天）；测试集为 2025 年（365 天）。三国使用相同日期划分。

## 变量与读取

NPZ 的 `targets` 和 `conditions` 形状均为 `(1744, 24, 3, 3)`，依次为天、小时、国家、变量。国家顺序为 DE、AT、FR。

`targets` 顺序为负荷、风电、光伏，单位均为 MW。`conditions` 顺序为 2 m 温度（°C）、100 m 风速（m/s）、向下短波辐射（W/m²）。CSV 对应字段为 load_mw、wind_mw、solar_mw、temperature_2m_c、wind_speed_100m_mps、shortwave_wm2。

这里提供物理单位数据，未执行新的标准化。训练中的标准化参数应只用训练集估计。NPZ 保留的装机容量及 CSV 的 wind_capacity_mw、solar_capacity_mw 是已有辅助字段，并非额外重新采集的数据。

```python
import numpy as np
with np.load('data/wind_pv_load_weather_2021_2025.npz', allow_pickle=False) as z:
    targets = z['targets']
    weather = z['conditions']
    splits = z['split']
    print(targets.shape, weather.shape, np.unique(splits, return_counts=True))
```

## 天气预报含义

NOAA/NCEP GFS 起报时刻为目标日前一天 00 UTC（source_init_utc）。瞬时字段使用 +24 至 +47 小时预报；辐射使用 +25 至 +48 小时区间均值重建小时值。它们是日前可获得的预报条件，不是目标日天气实测值。国家空间聚合使用网格与 NUTS 2016 level-0 边界的交叠面积。

## 缺失值和来源

最终 targets、conditions 均为有限值。原始追溯数组有意保留 NaN。实际风光荷以 ENTSO-E 为主，法国缺失值使用 RTE 补充，部分法国风电合并 ENTSO-E 陆上风电与 RTE 海上风电。法国 2021-10-31 00 UTC 的一个负荷值由相邻小时线性插值。

来源代码：-1 缺失；0 ENTSO-E；1 RTE；2 短缺口线性插值；3 ENTSO-E 陆上风电加 RTE 海上风电。CSV 的三个 source_code 字段分别对应负荷、风电、光伏。

数据来源：
- ENTSO-E: https://transparency.entsoe.eu/
- RTE: https://odre.opendatasoft.com/explore/dataset/eco2mix-national-cons-def/
- NOAA GFS: https://registry.opendata.aws/noaa-gfs-bdp-pds/
- GISCO NUTS: https://gisco-services.ec.europa.eu/distribution/v2/nuts/
- 装机容量来源 Energy-Charts: https://api.energy-charts.info/

行政边界署名：© EuroGeographics for the administrative boundaries.

## 数据内容与使用条款

本仓库提供德国、奥地利、法国处理后的风电、光伏和负荷数据、对应天气预报及训练、验证、测试集划分。

第三方数据适用原始提供方的使用条款。引用时请使用版本 v1.0.1：https://github.com/yml-0325/sc-resdiff-data/tree/v1.0.1 。

## English summary

This research snapshot contains hourly load, wind and PV observations and aligned day-ahead GFS weather conditions for Germany, Austria and France, from 24 March 2021 to 31 December 2025. All timestamps are UTC. Training/validation/test contain 1,013/366/365 days respectively. Arrays use day × hour × region × variable order. Targets are load/wind/solar in MW; weather variables are temperature (°C), wind speed (m/s) and shortwave radiation (W/m²). Final inputs contain no missing values; provenance arrays intentionally retain original missingness. The original NPZ is preserved byte-for-byte. Third-party data remain subject to the terms of their original providers.
