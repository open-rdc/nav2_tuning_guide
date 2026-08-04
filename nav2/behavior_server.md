# リカバリー・振る舞いサーバー (behavior_server)

ロボットが障害物に囲まれて動けなくなったり、経路が引けなくなった際に発動する「リカバリー動作（脱出動作）」を管理・実行するノードです。

## 基本パラメータ

`nav2_params.yaml` 内の `behavior_server` 名前空間で定義されています。

### 🌟 重要なパラメータ（調整の頻度：高）

*   **`recovery_plugins`** (現在値: ["backup", "wait"])
    *   意味: 使用するリカバリー動作の順番と種類を定義します。現在は「少し後ろに下がる（`backup`）」→「その場で待機して障害物が退くのを待つ（`wait`）」の順番に設定されています。
*   **`back_up.plugin`** / **`wait.plugin`**
    *   意味: それぞれのリカバリー動作で読み込むプラグイン名（`nav2_behaviors/BackUp`, `nav2_behaviors/Wait`）を指定します。
*   **`simulate_ahead_time`** (現在値: 2.0)
    *   意味: リカバリー動作を実行する前に、障害物と衝突しないかを先読みしてシミュレーションする時間(秒)です。
*   **`max_rotational_vel`** (現在値: 1.0) / **`min_rotational_vel`** (現在値: 0.4) / **`rotational_acc_lim`** (現在値: 3.2)
    *   意味: リカバリー時に回転動作を行う場合の、最大回転速度(rad/s)、最小回転速度(rad/s)、および回転加速度制限(rad/s^2)です。

### 🔧 その他の基本パラメータ

*   **`costmap_topic`** (現在値: local_costmap/costmap_raw)
    *   意味: 障害物判定のために参照するローカルコストマップのトピック名です。
*   **`footprint_topic`** (現在値: local_costmap/published_footprint)
    *   意味: ロボットの形状（フットプリント）を参照するためのトピック名です。
*   **`cycle_frequency`** (現在値: 10.0)
    *   意味: サーバーがリカバリー動作を管理・実行する周波数(Hz)です。
*   **`global_frame`** (現在値: odom)
    *   意味: 動作の基準となるグローバル座標系です。
*   **`robot_base_frame`** (現在値: base_link)
    *   意味: ロボットの基準となる座標系です。
*   **`transform_timeout`** (現在値: 0.1)
    *   意味: tf（座標変換）の取得を待機するタイムアウト時間(秒)です。
*   **`use_sim_time`** (現在値: false)
    *   意味: シミュレーション時間を使用するかどうか。実機の場合は `false` にします。