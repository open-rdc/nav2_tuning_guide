# emcl2_ros2 (拡張モンテカルロ位置推定)

## 概要

emcl2_ros2 は、ROS 2 環境での自己位置推定を行うパッケージです。ロボットが自分の位置を推定するために使用されます。

## 自己位置推定とは？

自己位置推定（Self-Localization）とは、ロボットが周辺環境における自分の現在位置を認識するプロセスです。LiDAR センサーからの距離データ `/scan` と事前に用意した地図を照合することにより、ロボットの位置を推定します。

## なぜ AMCL ではなく emcl2 を使うのか？

- **AMCL（Adaptive Monte Carlo Localization）**: 古いアルゴリズムで計算量が多い
- **emcl2（Expansion Monte Carlo Localization 2）**: より効率的で、ロボットの位置が大きく異なる場合でも迅速に復帰できる改善版アルゴリズム

emcl2 は AMCL より高速で精度が高いため、実運用環境での使用が推奨されます。

---

## セットアップ

### 環境
- **OS**: Ubuntu 22.04 LTS
- **ROS 2**: Humble

### 必要なパッケージのインストール

#### 1. rviz2（可視化ツール）
```bash
$ sudo apt install ros-humble-rviz2
```

#### 2. emcl2_ros2
```bash
$ git clone https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2.git
```

#### 3. その他の必須パッケージ
```bash
$ sudo apt install ros-humble-nav2-lifecycle-manager
$ sudo apt install ros-humble-nav2-map-server
```

---

## 起動手順

### 1. パラメータファイルの準備

emcl2_ros2 のパラメータファイル：
https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2/blob/main/config/emcl2.param.yaml

**重要**: 起動前にパラメータファイル内の以下の部分を修正してください：
```yaml
# 変更前
footprint_frame_id: "base_footprint"

# 変更後
footprint_frame_id: "base_link"
```

### 2. rosbag の再生（必要な Topic のみ）

```bash
$ ros2 bag play rosbag2_2026_01_31-16_40_55 --clock --topics /odometry/filtered /tf /tf_static /scan
```

### 3. emcl2 の起動

```bash
$ ros2 launch emcl2 emcl2.launch.py map:=/path/to/your/map.yaml use_sim_time:=true
```

**例** (orne-box の場合):
```bash
$ ros2 launch emcl2 emcl2.launch.py map:=/home/rosuser/box3_ws/src/orne-box/orne_box_navigation_executor/config/maps/cit_3f_map.yaml use_sim_time:=true
```

**注意**: `cit_3f_map.yaml` のパスは各自の環境に合わせて変更してください。
- `cit_3f_map.yaml` と `cit_3f_map.pgm` はペアで必要です

**地図ファイルの参照先** (orne-box):
https://github.com/open-rdc/orne-box/tree/humble-devel/orne_box_navigation_executor/config/maps

### 4. rviz2 の起動

```bash
$ rviz2
```

#### 起動時の様子

**ROS 2 Humble の場合**:
- Map が表示されないことがありますが、一旦無視して進めてください（可視化できていないだけで、調整は可能です）
- 参考動画: https://youtu.be/JVHlCpinnRE

**ROS 1 Noetic の場合** (参考):
- こちらでは Map が表示できます
- 参考動画: https://youtu.be/frq4eJWPrHk

---

## 可視化（RViz2 での表示設定）

rviz2 で以下の Topic を可視化することで、位置推定の状態を確認できます

### 可視化する Topic
- `/map` - 作成した地図
- `/mcl_pose` - 推定位置（平均の位置）
- `/particlecloud` - パーティクルの分布（多数の仮説位置）
- `/scan` - LiDAR スキャンデータ

### 追加方法

1. RViz2 左下の赤枠の **Add** ボタンを押す
2. Topic 一覧から目的の Topic を選択
3. **OK** を押して追加

複数の Topic を同様に追加することで、位置推定の動作を可視的に確認できます。

---

## パラメータ調整

emcl2 の精度や動作は、パラメータファイルで調整できます

**パラメータファイル**:
https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2/blob/main/config/emcl2.param.yaml

詳細な調整方法については、別ファイル（emcl2_params.md）を参照してください。

---

## トラブルシューティング

### rosbag の中身を確認したい場合

```bash
$ ros2 bag info rosbag2_2026_01_31-16_40_55
```

---

## 参考リンク

- **emcl2_ros2 GitHub**: https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2
- **orne-box**: https://github.com/open-rdc/orne-box

