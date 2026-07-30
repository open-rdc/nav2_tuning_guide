# emcl2_ros2

## 概要

自己位置推定（Self-Localization）とは、ロボットが周辺環境における自分の現在位置を認識するプロセスです。LiDAR センサーからの距離データと事前に用意した地図を照合することにより、ロボットの位置を推定します。

---

## 環境

- **OS**：Ubuntu 22.04 LTS
- **ROS 2**：Humble

---

## 1. 必要なパッケージの準備

### RViz2

```bash
sudo apt install ros-humble-rviz2
```

### emcl2_ros2

```bash
cd ~/ros2_ws/src
git clone https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2.git
```

### その他のパッケージ

```bash
sudo apt install ros-humble-nav2-lifecycle-manager
sudo apt install ros-humble-nav2-map-server
```

### ビルド

```bash
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

（新しい端末を開いた場合は `source ~/ros2_ws/install/setup.bash` を実行してください）

---

## 2. パラメータファイルの確認

使用するパラメータファイル:

- [emcl2.param.yaml](https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2/blob/main/config/emcl2.param.yaml)

### Box3（orne-box）の例

Box3/orne-box の TF 構成に合わせて `footprint_frame_id` を変更する必要があります。

**重要**: 起動前にパラメータファイル内の以下の部分を確認・修正してください。

**注意**: 下の変更例は orne-box の例です。`footprint_frame_id` は実際にお使いのロボットで使用している TF フレーム名に合わせて変更してください（orne-box では `base_link` を使います）。

```yaml
# 変更前
footprint_frame_id: "base_footprint"

# 変更後
footprint_frame_id: "base_link"
```

### rosbag の再生（必要な Topic のみ）

```bash
$ ros2 bag play <your_rosbag> --clock --topics /odometry/filtered /tf /tf_static /scan
```

### emcl2 の起動

```bash
$ ros2 launch emcl2 emcl2.launch.py map:=/path/to/your/map.yaml use_sim_time:=true
```

**例** (orne-box の場合):

```bash
$ ros2 launch emcl2 emcl2.launch.py map:=/home/rosuser/box3_ws/src/orne-box/orne_box_navigation_executor/config/maps/cit_3f_map.yaml use_sim_time:=true
```

---

## 3. rosbag の中身を確認

使用する rosbag に必要な Topic が記録されているか確認します。

```bash
ros2 bag info <your_rosbag>
```

少なくとも、次の Topic が必要です。

- `/odometry/filtered`
- `/tf`
- `/tf_static`
- `/scan`

![rosbagの中身の確認](images/rosbag_info.png)

> [!NOTE]
>
> rosbag のファイル名は、使用するものに変更してください。

詳細な調整方法については、別ファイル（[emcl2_1.md](emcl2_1.md) と [emcl2_2.md](emcl2_2.md)）を参照してください。

---

## 4. 地図ファイルの準備

自己位置推定には、事前に作成した地図が必要です。

通常、次の2つのファイルをセットで使用します。

```text
map.yaml
map.pgm
```

Box3で使用している地図は、次のディレクトリにあります。

```text
orne_box_navigation_executor/config/maps/
```

- [Box3の地図ファイル](https://github.com/open-rdc/orne-box/tree/humble-devel/orne_box_navigation_executor/config/maps)

---

## 5. 起動手順

端末を3つ使用します。

### 端末1：rosbagを再生

```bash
ros2 bag play <your_rosbag> --clock --topics /odometry/filtered /tf /tf_static /scan
```

### 端末2：emcl2_ros2を起動

```bash
ros2 launch emcl2 emcl2.launch.py map:=/path/to/your/map.yaml use_sim_time:=true
```

### 端末3：RViz2を起動

```bash
rviz2
```

---

## 6. RViz2の表示設定

Fixed Frame を `map` に設定し、次の Topic を追加してください： `/map`, `/scan`, `/particlecloud`, `/mcl_pose`。

---

## 7. 初期位置の設定

RViz2 の `2D Pose Estimate` を使って初期位置を設定し、パーティクルの収束を確認してください。

---

## 8. 正常に動作しているかの確認

- 地図がRViz2に表示されている
- LaserScanと地図が大きくずれていない
- パーティクルがロボット周辺に収束している

---

## 9. パラメータ調整へ進む

- [オドメトリ誤差パラメータの調整](./emcl2_1.md)
- [膨張リセットとその他のパラメータの調整](./emcl2_2.md)

---

## トラブルシューティング

- `map:=` に指定したパスが正しいか
- `.yaml` と `.pgm` の両方が存在するか
- RViz2 の `Fixed Frame` が `map` になっているか

---

## 参考リンク

- [emcl2_ros2](https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2)
- [emcl2_ros2 のパラメータファイル](https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2/blob/main/config/emcl2.param.yaml)
- [orne-box の地図ファイル](https://github.com/open-rdc/orne-box/tree/humble-devel/orne_box_navigation_executor/config/maps)
