# Costmapのパラメータ

`Costmap` は，ロボットの周囲環境をコストとして表現する重要な地図である．  
Nav2では，地図情報やLiDARなどのセンサ情報をもとに，障害物領域および障害物周辺の危険領域を表現する．  
デフォルト設定でも動作するが，障害物回避の精度や動的障害物に対する反応性を高めたい場合には，パラメータの調整が有効である．

## パラメータの説明（[参考：Nav2 Costmap 2D](https://docs.nav2.org/configuration/packages/configuring-costmaps.html)）

Nav2では，主に `local_costmap` と `global_costmap` の2種類のCostmapが使用される．  
`local_costmap` はロボット周辺の障害物情報を扱い，主に経路追従や障害物回避に使用される．  
`global_costmap` は地図全体の障害物情報を扱い，主に目標地点までの経路計画に使用される．

本ページでは，orne-boxで使用されている以下のパラメータファイルに記述されたCostmap関連パラメータを対象とする．

```text
orne_box_navigation_executor/config/params/nav2_params.yaml
```

### local_costmap

#### `update_frequency`

- **意味**: Costmapを更新する周波数（Hz） *(default: 5.0)*

#### `width`

- **意味**: Local Costmapの横幅（m） *(default: 5)*

#### `height`

- **意味**: Local Costmapの縦幅（m） *(default: 5)*

#### `inflation_radius`

- **意味**: 障害物の周囲にコストを広げる半径（m） *(default: 0.55)*

#### `cost_scaling_factor`

- **意味**: 障害物から離れるにつれて，膨張したコストが減少する度合いを決める係数 *(default: 10.0)*

#### `resolution`

- **意味**: Costmapの1セルが表す実空間上の大きさ（m/cell） *(default: 0.1)*

#### `obstacle_max_range`

- **意味**: センサ観測を障害物としてCostmapに反映する最大距離（m） *(default: 2.5)*

#### `raytrace_max_range`

- **意味**: センサ観測に基づいて障害物をCostmapから消去する最大距離（m） *(default: 3.0)*

---

### global_costmap

#### `update_frequency`

- **意味**: Costmapを更新する周波数（Hz） *(default: 5.0)*

#### `inflation_radius`

- **意味**: 障害物の周囲にコストを広げる半径（m） *(default: 0.55)*

#### `cost_scaling_factor`

- **意味**: 障害物から離れるにつれて，膨張したコストが減少する度合いを決める係数 *(default: 10.0)*

#### `resolution`

- **意味**: Costmapの1セルが表す実空間上の大きさ（m/cell） *(default: 0.1)*

#### `obstacle_max_range`

- **意味**: センサ観測を障害物としてCostmapに反映する最大距離（m） *(default: 2.5)*

#### `raytrace_max_range`

- **意味**: センサ観測に基づいて障害物をCostmapから消去する最大距離（m） *(default: 3.0)*