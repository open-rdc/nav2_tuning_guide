# Nav2

## 本章の目的

本章では，Nav2を用いた自律移動に必要な構成要素を整理する．

Nav2は，地図，自己位置推定，センサ情報をもとに，経路計画，経路追従，障害物回避，waypoint走行を行うためのナビゲーションフレームワークである．


## 構成

| 項目 | 内容 |
| :--- | :--- |
| costmap | 地図やセンサ情報をもとに通行可能領域を表現する |
| planning | 現在位置から目標位置までの経路を生成する |
| control | 生成された経路に沿ってロボットを走行させる |
| waypoint | 複数の目標地点を順番に走行する |

## 目次

### Costmap（コストマップ）
* [global_costmap.md](global_costmap.md): 全体地図に基づくグローバルコストマップ
* [local_costmap.md](local_costmap.md): センサ情報に基づく周辺ローカルコストマップ

### Planning（経路計画・全体管理）
* [planner_server.md](planner_server.md): プランナーサーバー（全体経路生成）
* [bt_navigator.md](bt_navigator.md): ビヘイビアツリー（ナビゲーション全体の頭脳）

### Control（経路追従・制御・リカバリー）
* [controller_server.md](controller_server.md): コントローラサーバー（軌道追従・障害物回避）
* [smoother_server.md](smoother_server.md): パス平滑化サーバー
* [velocity_smoother.md](velocity_smoother.md): 速度平滑化ノード（急発進・急ブレーキ防止）
* [prestop_node.md](prestop_node.md): 衝突防止・安全停止ノード
* [behavior_server.md](behavior_server.md): リカバリー・振る舞いサーバー（脱出動作など）

### Waypoint（ウェイポイント走行）
* [waypoint_follower.md](waypoint_follower.md): ウェイポイント追従サーバー