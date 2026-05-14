# nav2_tuning_guide

ロボットのナビゲーションを調整するための、ROS 2 (Navigation2 / Nav2) 向けパラメータ調整・トラブルシューティングドキュメントです。
以下の一部ファイル名およびコマンドは、[orne-box](https://github.com/open-rdc/orne-box) の ROS 2 ブランチ環境を基準にしています。

## 目次

* [1. Nav2 アーキテクチャの基礎と調整アプローチ](#1-nav2-アーキテクチャの基礎と調整アプローチ)
* [2. 地図作成](#2-地図作成)
  * [slam_toolbox (屋内2Dマップ作成)](#slam_toolbox-屋内2Dマップ作成)
  * [glim + pointcloud2pgm_slicer (屋外向けマップ作成)](#glim--pointcloud2pgm_slicer-屋外向けマップ作成)
* [3. icart (オドメトリ調整)](#3-icart-オドメトリ調整)
* [4. AMCL (自己位置推定)](#4-amcl-自己位置推定)
* [5. Nav2 パラメータ調整](#5-nav2-パラメータ調整)
  * Costmap2D
  * Planner / Controller
  * Behavior Tree (リカバリー行動)
* [6. 問題の切り分けとトラブルシューティング](#6-問題の切り分けとトラブルシューティング)

---

## 2. 地図作成

自律移動において、正確な環境地図の構築はすべての前提となります。ここでは環境に応じた2種類のマッピング手法を解説します。

### slam_toolbox (屋内2Dマップ作成)

研究室や廊下など、比較的平坦で特徴点が捉えやすい屋内環境では、2D LiDARとオドメトリを利用する `slam_toolbox` によるマッピングが有効です。

#### 2.1. マッピングの実行手順

1. **ロボットの起動**
   `orne_box` を起動し、センサデータとオドメトリが正常に出力されていることを確認します。
   ```bash
   ros2 launch orne_box_bringup orne_box_bringup.launch.py