# Costmapのパラメータ

## 1. Costmapの役割

Costmapは，ロボットが移動する環境をコストとして表現した地図である．

Nav2では，事前に作成した地図やLiDARなどのセンサ情報をもとに，
通行可能な領域，障害物領域，および障害物周辺の危険領域を表現する．

Costmapは，経路計画および経路追従・障害物回避に用いられるため，
Nav2の調整において重要な要素である．

## 2. 確認対象パラメータ

本ページでは，Nav2のCostmapに関する以下のパラメータについて，
値を変更した場合の影響を確認する．

| パラメータ | 主な確認内容 |
| --- | --- |
| `update_frequency` | Costmapの更新周期 |
| `width` | Local Costmapの横幅 |
| `height` | Local Costmapの縦幅 |
| `resolution` | Costmapの解像度 |
| `inflation_radius` | 障害物周辺の膨張範囲 |
| `cost_scaling_factor` | 障害物周辺のコスト分布 |
| `obstacle_max_range` | 障害物を反映する最大距離 |
| `raytrace_max_range` | 障害物を消去する最大距離 |

## 3. orne-boxで使用されるパラメータファイル

orne-boxでNav2を使用する際の主なパラメータは，以下のファイルに記述されている．

```text
orne_box_navigation_executor/config/params/nav2_params.yaml