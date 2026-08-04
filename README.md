# nav2_tuning_guide

ロボットのナビゲーションを調整するためのドキュメント。以下の一部ファイル名およびコマンドは、[orne-box](https://github.com/open-rdc/orne-box)のROS 2環境を基準にしています。

## 目次

* [パッケージ別パラメータ調整](#パッケージ別パラメータ調整)
  * [icart（オドメトリ調整）](#icart-オドメトリ調整)
  * [emcl2_ros2（自己位置推定）](#emcl2_ros2-自己位置推定)
  * [Nav2（経路計画・ナビゲーション）](#nav2-経路計画ナビゲーション)
* [地図作成](#地図作成)

---

## パッケージ別パラメータ調整

### icart（オドメトリ調整）

まずオドメトリの調整を。

* [icart_1.md](icart/icart_1.md): オドメトリ調整の手順

### emcl2_ros2（自己位置推定）

rosbagを活用してパラメータを調整します。

* [README.md](emcl2_ros2/README.md): emcl2_ros2の基本設定
* [emcl2_1.md](emcl2_ros2/emcl2_1.md): 主要パラメータの調整
* [emcl2_2.md](emcl2_ros2/emcl2_2.md): その他パラメータの調整

### Nav2（経路計画・ナビゲーション）

Nav2の各サーバー・ノードの要素の調整と設定。

* [global_costmap.md](nav2/global_costmap.md): グローバルコストマップのパラメータ調整
* [local_costmap.md](nav2/local_costmap.md): ローカルコストマップのパラメータ調整
* [planner_server.md](nav2/planner_server.md): プランナーサーバー（全体経路生成）のパラメータ調整
* [controller_server.md](nav2/controller_server.md): コントローラサーバー（軌道追従・障害物回避）のパラメータ調整
* [behavior_server.md](nav2/behavior_server.md): リカバリー・振る舞いサーバー（脱出動作）のパラメータ調整
* [smoother_server.md](nav2/smoother_server.md): パス平滑化サーバーのパラメータ調整
* [bt_navigator.md](nav2/bt_navigator.md): ビヘイビアツリー（ナビゲーション全体の頭脳）のパラメータ調整
* [waypoint_follower.md](nav2/waypoint_follower.md): ウェイポイント追従サーバーのパラメータ調整
* [velocity_smoother.md](nav2/velocity_smoother.md): 速度平滑化ノード（急発進・急ブレーキ防止）のパラメータ調整
* [prestop_node.md](nav2/prestop_node.md): 衝突防止・安全停止ノードのパラメータ調整

---

## 地図作成

地図作成の手順とパラメータ。

* [map_1.md](map/map_1.md): 地図作成方法に関して
* [slam_toolbox.md](map/slam_toolbox.md): slam_toolboxでの地図作成方法
