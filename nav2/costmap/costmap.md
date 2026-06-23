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

## 実機Box3でのCostmap調整ポイント

1. **まずは可視化で確認**
   - RVizで `local_costmap/costmap` と `global_costmap/costmap` を表示し、障害物やinflationが正しく反映されているか確認する。
   - センサデータがCostmapに入っていない場合は、`obstacle_layer` やトピック名、フレーム設定を確認する。

2. **Box3の実寸・安全マージンに合わせる**
   - `robot_radius` や `footprint` に基づき、`inflation_radius` を実機サイズ + 余裕分に設定する。
   - 例: 実際のロボット半径が0.25mなら、`inflation_radius` は0.35〜0.45m程度を試す。

3. **コスト分布の調整**
   - `cost_scaling_factor` を低めにするとObstacle周辺のコストが緩やかになり、狭い箇所で通りやすくなる。
   - 逆に高くすると危険領域が急峻になり、安全性は上がるが旋回や狭所通過で停滞しやすくなる。

4. **範囲と解像度のバランス**
   - `resolution` を細かくすると精度は上がるが計算負荷が増える。
   - Box3では `0.1` か `0.05` を基準に、CPU負荷と反応性を見ながら調整する。

5. **実機テスト時のワークフロー**
   - `nav2_bringup` を立ち上げ、RVizでCostmapの表示とトピックを確認。
   - 走行中に `inflation_radius` / `cost_scaling_factor` / `obstacle_max_range` を少しずつ変更し、再起動後に挙動を比較する。
   - 必要なら `local_costmap -> width/height` を広げて、旋回や回避の予測領域を確保する。

6. **よくある「実機で走らない」原因**
   - センサデータがCostmapに反映されていない
   - フットプリントとinflationの設定が不適切で通路を塞いでいる
   - `obstacle_max_range` が短すぎて遠方障害物を認識できない
   - `raytrace_max_range` が短すぎて消去が追いつかない

## 参考: 実機で試すべき優先パラメータ

- `local_costmap/inflation_radius`
- `local_costmap/cost_scaling_factor`
- `local_costmap/obstacle_max_range`
- `local_costmap/raytrace_max_range`
- `local_costmap/width`, `local_costmap/height`
- `local_costmap/resolution`

Box3を走らせるときは、まずこのあたりを中心に変更し、RVizでCostmapの見え方とロボット挙動を同時に確認してください。

## 実機でCostmapパラメータを変更する手順

1. **編集するファイルを確認する**
   - 実機Box3のNav2設定は `orne-box/orne_box_navigation_executor/config/params/nav2_params.yaml` にあります。
   - `nav2_params_override.yaml` は追加変更/オーバーライド用です。

2. **まずは実機で動いている現在値を確認する**
   - RVizで `local_costmap/costmap` と `global_costmap/costmap` を表示。
   - 障害物が正しくマークされているか、inflationが広がりすぎていないかを見る。

3. **優先して変更すべきパラメータ**
   a. `local_costmap -> inflation_layer -> inflation_radius`
   b. `local_costmap -> inflation_layer -> cost_scaling_factor`
   c. `local_costmap -> obstacle_layer -> scan -> obstacle_max_range`
   d. `local_costmap -> obstacle_layer -> scan -> raytrace_max_range`
   e. `local_costmap -> width` / `height`
   f. `local_costmap -> resolution`
   g. 必要なら `global_costmap -> inflation_layer -> inflation_radius` と `global_costmap -> inflation_layer -> cost_scaling_factor`

4. **実機変更の基本パターン**
   - まずは `local_costmap/inflation_radius` を調整し、安全マージンを確認。
   - `cost_scaling_factor` を変更し、障害物周辺のコストの広がり方を見比べる。
   - `obstacle_max_range` が短いと遠くの障害物を見落とすので、`scan` の最大値は実機のLiDAR性能に合わせる。
   - `raytrace_max_range` が短すぎると障害物除去が追いつかず、古い障害物情報が残る。

5. **変更方法**
   - `orne-box/orne_box_navigation_executor/config/params/nav2_params.yaml` をテキストエディタで開く。
   - `local_costmap:` と `global_costmap:` セクションを探す。
   - 変更したい値を書き換える。
   - 保存してから Nav2 を再起動する。

6. **Box3実機での最初の試行値例**
   - `local_costmap/inflation_radius`: 0.3
   - `local_costmap/cost_scaling_factor`: 3.0
   - `local_costmap/obstacle_layer/scan/obstacle_max_range`: 3.5
   - `local_costmap/obstacle_layer/scan/raytrace_max_range`: 4.0
   - `local_costmap/width`: 10
   - `local_costmap/height`: 10
   - `local_costmap/resolution`: 0.1

7. **変更後の確認**
   - 変更後に Nav2 を起動し、RViz で Costmap 表示を確認。
   - 同じルートを走らせ、回避性能や狭所通過の挙動を比較する。
   - `inflation_radius` を大きくしすぎると通路が狭く見えるので、実際に通れるかを必ず確認。

## `orne-box` の実際の参考値

`orne-box/orne_box_navigation_executor/config/params/nav2_params.yaml` に書かれている実機例は以下の通りです。

- local_costmap
  - `update_frequency`: 4.0505
  - `width`: 10
  - `height`: 10
  - `resolution`: 0.1
  - `inflation_radius`: 0.3
  - `cost_scaling_factor`: 3.0
  - `obstacle_max_range`: 3.5
  - `raytrace_max_range`: 4.0

- global_costmap
  - `update_frequency`: 2.55055
  - `resolution`: 0.1
  - `inflation_radius`: 1.25
  - `cost_scaling_factor`: 3.0
  - `obstacle_max_range`: 10.0
  - `raytrace_max_range`: 11.0

これらをベースに、Box3特有の狭い通路や机の脚がある現場での実走行結果を記録してください。

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