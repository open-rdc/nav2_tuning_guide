# emcl2_ros2 

## 概要

emcl2_ros2 は、ROS 2 環境での自己位置推定を行うパッケージです。ロボットが自分の位置を推定するために使用されます。

## 自己位置推定とは？

自己位置推定（Self-Localization）とは、ロボットが周辺環境における自分の現在位置を認識するプロセスです。LiDAR センサーからの距離データと事前に用意した地図を照合することにより、ロボットの位置を推定します。



---


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

**重要**: 起動前にパラメータファイル内の以下の部分を修正してください。

**注意**: 下の変更例は orne-box の一例です。`footprint_frame_id` は実際にお使いのロボットで使用している TF フレーム名に合わせて変更してください（orne-box では `base_link` を使います）。
```yaml
# 変更前
footprint_frame_id: "base_footprint"

# 変更後
footprint_frame_id: "base_link"
```

### 2. rosbag の再生（必要な Topic のみ）

```bash
$ ros2 bag play <your_rosbag> --clock --topics /odometry/filtered /tf /tf_static /scan
```

### 3. emcl2 の起動

```bash
$ ros2 launch emcl2 emcl2.launch.py map:=/path/to/your/map.yaml use_sim_time:=true
```

**例** (orne-box の場合):
```bash
$ ros2 launch emcl2 emcl2.launch.py map:=/home/rosuser/box3_ws/src/orne-box/orne_box_navigation_executor/config/maps/cit_3f_map.yaml use_sim_time:=true
```



### 4. rviz2 の起動

```bash
$ rviz2
```



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

詳細な調整方法については、別ファイル（[emcl2_1.md](emcl2_1.md) と [emcl2_2.md](emcl2_2.md)）を参照してください。

---

## トラブルシューティング

### rosbag の中身を確認したい場合

```bash
$ ros2 bag info <your_rosbag>
```

---

## 参考リンク

- **emcl2_ros2 GitHub**: https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2


