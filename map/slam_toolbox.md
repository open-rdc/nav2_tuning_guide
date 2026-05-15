# slam_toolboxを用いた地図作成

## 概要

本ページでは，実機で取得したrosbagを用いて，slam_toolboxにより2D地図を作成する手順を示す．  
作成した地図は，Nav2で使用する地図ファイルとして保存する．

---

## 使用するもの

- 実機で取得したrosbag
- slam_toolbox
- nav2_map_server

---

## 1. slam_toolboxのインストール

slam_toolboxをインストールする．

```bash
sudo apt install ros-humble-slam-toolbox
```

地図の保存には `nav2_map_server` を使用する．

```bash
sudo apt install ros-humble-nav2-map-server
```

---

## 2. rosbagの確認

取得したrosbagに，地図作成に必要なトピックが含まれているか確認する．

```bash
ros2 bag info <rosbag名>
```

主に確認するトピックは以下である．

| トピック | 内容 |
|---|---|
| `/scan` | LiDARのLaserScanデータ |
| `/tf` | 座標変換情報 |
| `/tf_static` | 静的な座標変換情報 |
| `/odom` | オドメトリ情報 |

<!-- ここに ros2 bag info のスクリーンショットを入れる -->


---

## 3. slam_toolbox用パラメータファイル

slam_toolboxの起動には，パラメータファイルを使用する．  
以下は，今回使用したパラメータの一部である．

```yaml
slam_toolbox:
  ros__parameters:
    odom_frame: odom
    map_frame: map
    base_frame: base_link
    scan_topic: /scan
    mode: mapping
    resolution: 0.1
    min_laser_range: 0.5
    max_laser_range: 28.0
    use_scan_matching: true
    do_loop_closing: true
```

主に確認する項目は以下である．

| パラメータ | 内容 |
|---|---|
| `odom_frame` | オドメトリ座標系 |
| `map_frame` | 地図座標系 |
| `base_frame` | ロボット本体の座標系 |
| `scan_topic` | slam_toolboxに入力するLaserScanトピック |
| `mode` | 地図作成を行うため `mapping` に設定する |
| `resolution` | 作成する地図の解像度 |

<!-- ここに slam_toolbox_params.yaml のスクリーンショットを入れる -->
[slam_toolbox_params.yaml](./slam_toolbox_params.yaml)

---

## 4. slam_toolboxの起動

以下のコマンドでslam_toolboxを起動する．

```bash
ros2 launch slam_toolbox online_async_launch.py slam_params_file:=<slam_toolbox_params.yamlのパス>
```

---

## 5. rosbagの再生

別のターミナルでrosbagを再生する．

```bash
ros2 bag play <rosbag名> --clock --topics /scan /tf /tf_static
```

`--clock` を付けることで，rosbagに記録された時刻を用いて再生する．

---

## 6. RViz2で地図を確認

RViz2を起動する．

```bash
rviz2
```

RViz2では以下のように設定する．

1. `Fixed Frame` を `map` にする
2. `Add` を選択する
3. `By Topic` から `/map` を追加する

正常に地図が作成されると，RViz2上に地図が表示される．

<!-- ここに RViz2で /map が表示されているスクリーンショットを入れる -->
![RViz2で作成中の地図を確認](../images/slam_toolbox/rviz_map_display.png)

---

## 7. 地図の保存

地図が作成できたら，slam_toolboxを起動したまま，以下のコマンドで地図を保存する．

```bash
ros2 run nav2_map_server map_saver_cli -f <保存先>/<地図名>
```

保存に成功すると，以下の2つのファイルが作成される．

```text
<地図名>.pgm
<地図名>.yaml
```

`.pgm` は地図画像，`.yaml` は地図の解像度や原点などを記述した設定ファイルである．

<!-- ここに保存された地図画像を入れる -->
![作成した地図](../images/slam_toolbox/generated_map.png)

---

## 8. 作成した地図の使用先

作成した地図は，以下の工程で使用する．

- オフライン自己位置推定
- costmap用地図の作成
- waypoint配置
- 実機でのNav2走行

---

## まとめ

slam_toolboxを用いることで，box3で取得したrosbagから2D地図を作成できた．  
作成した地図は `.pgm` と `.yaml` として保存し，後続のNav2走行手順で使用する．