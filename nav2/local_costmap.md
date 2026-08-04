# ローカルコストマップ (Local Costmap)

## 概要
ローカルコストマップは、ロボット周辺の短期的な障害物情報を保持し、経路追従や衝突回避（コントローラ）に使用されます。センサーからの最新の観測を反映するため、更新頻度を高めに設定するのが一般的です。

## 主なパラメータ
- `update_frequency` / `publish_frequency`: センサーデータの取り込みと公開頻度
- `rolling_window`: ロボットを中心にウィンドウを移動するか（通常 true）
- `resolution`: 解像度（例: 0.05–0.10 m）
- `width` / `height`: ローカルマップの幅・高さ（メートル）
- `inflation_radius`: ロボット周辺の膨張半径（クリアランス）
- `obstacle_range` / `raytrace_range`: センサーで扱う距離範囲
- レイヤー: `obstacle_layer`, `inflation_layer`, 必要であれば `voxel_layer` など

## 推奨設定例（屋内ロボット）
- `resolution`: 0.05–0.10
- `width` / `height`: 6.0（状況に応じて）
- `rolling_window`: true
- `inflation_radius`: 0.3–0.5

## 注意点
- センサー座標系とTFが正しく設定されていることを確認してください。TFのずれは障害物未検出や誤った位置情報の原因になります。
- 更新頻度が低いと障害物情報が古くなり、衝突リスクが増します。
- 膨張（inflation）は経路の安全性に直結するため、グローバルコストマップと整合させてください。

## トラブルシューティング
- 障害物が検出されない: センサーのトピックや高さ制限（min/max obstacle height）を確認。
- コストマップが空になる: `rolling_window` や origin の設定、TF遅延を確認。
- 経路が狭くなる/通れない: `inflation_radius` や `cost_scaling_factor` を見直す。
