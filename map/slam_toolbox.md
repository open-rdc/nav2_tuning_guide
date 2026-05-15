# slam_toolboxを用いた地図作成

## 目的

本章では，ROS 2 Humble環境において，box3で取得したrosbagを用いてslam_toolboxにより2D地図を作成する手順を示す．

作成した地図は，Nav2のmap_serverで読み込まれ，自己位置推定や経路計画に使用される．  
そのため，地図作成はNav2で安定して自律移動を行うための前処理として重要である．

---

## 使用環境

| 項目 | 内容 |
|---|---|
| ロボット | box3 |
| OS | Ubuntu 22.04 |
| ROS 2 | Humble |
| 使用パッケージ | slam_toolbox, nav2_map_server |
| 使用データ | 津田沼チャレンジコースで取得したrosbag |
| 地図作成方法 | rosbagを用いたオフライン地図作成 |

---

## 全体の流れ

本手順では，以下の流れで地図を作成する．

1. rosbagに必要なトピックが含まれているか確認する
2. slam_toolbox用のパラメータファイルを用意する
3. slam_toolboxを起動する
4. rosbagを再生する
5. RViz2で `/map` を確認する
6. 作成された地図を保存する
7. `.pgm` と `.yaml` が生成されたことを確認する

---

## 1. rosbagの確認

まず，取得済みrosbagに地図作成に必要なトピックが含まれているか確認する．

```bash
cd ~/bags
ros2 bag info rosbag2_2026_01_31-16_40_55
```

今回使用したrosbagには，以下のようなトピックが含まれていた．

```text
/scan
/tf
/tf_static
/odom
/odometry/filtered
/imu/data
/clock
```

slam_toolboxで地図作成を行う際，主に必要となるトピックは以下である．

| トピック | 役割 |
|---|---|
| `/scan` | LiDARのLaserScanデータ |
| `/tf` | 動的な座標変換 |
| `/tf_static` | 静的な座標変換 |
| `/odom` | オドメトリ情報 |
| `/clock` | rosbag再生時の時刻 |

> **画像を入れる場所**  
> ここに `ros2 bag info` の実行結果のスクリーンショットを貼る．  
> 例：`images/rosbag_info.png`

```markdown
![rosbag infoの確認](images/rosbag_info.png)
```

---

## 2. slam_toolboxのインストール

slam_toolboxを使用するため，以下のコマンドでインストールする．

```bash
sudo apt install ros-humble-slam-toolbox
```

また，地図保存には `nav2_map_server` の `map_saver_cli` を使用する．

```bash
sudo apt install ros-humble-nav2-map-server
```

---

## 3. slam_toolbox用パラメータファイルの確認

slam_toolboxを起動するために，`slam_toolbox_params.yaml` を使用する．

今回使用したパラメータファイルでは，以下の設定を確認した．

```bash
cd ~/bags
grep -n "use_sim_time\|odom_frame\|map_frame\|base_frame\|scan_topic" slam_toolbox_params.yaml
```

確認結果の例を以下に示す．

```yaml
odom_frame: odom
map_frame: map
base_frame: base_link
scan_topic: /scan
```

各パラメータの意味は以下である．

| パラメータ | 意味 |
|---|---|
| `odom_frame` | オドメトリ座標系 |
| `map_frame` | 地図座標系 |
| `base_frame` | ロボット本体の座標系 |
| `scan_topic` | slam_toolboxに入力するLaserScanトピック |

rosbagを `--clock` 付きで再生する場合は，slam_toolbox側で `use_sim_time: true` を設定する必要がある．  
設定されていない場合，rosbag内の時刻とslam_toolboxの時刻が合わず，TF取得に失敗することがある．

```yaml
use_sim_time: true
```

> **画像を入れる場所**  
> ここに `slam_toolbox_params.yaml` の中身，または `grep` 結果のスクリーンショットを貼る．  
> 例：`images/slam_toolbox_params.png`

```markdown
![slam_toolbox用パラメータの確認](images/slam_toolbox_params.png)
```

---

## 4. slam_toolboxの起動

以下のコマンドでslam_toolboxを起動する．

```bash
cd ~/bags
ros2 launch slam_toolbox online_async_launch.py slam_params_file:=/home/sakai/bags/slam_toolbox_params.yaml
```

このとき，slam_toolboxは `/scan`，`/tf`，`/tf_static` などを用いて地図を生成する．

---

## 5. rosbagの再生

別ターミナルを開き，rosbagを再生する．

```bash
cd ~/bags
ros2 bag play rosbag2_2026_01_31-16_40_55 --clock --topics /scan /tf /tf_static
```

`--clock` を付けることで，rosbagに記録された時刻を使用して再生できる．

もしslam_toolbox側で以下のような警告が出る場合がある．

```text
[slam_toolbox]: Failed to compute odom pose
```

この場合，slam_toolboxが `odom` から `base_link` へのTFを取得できていない可能性がある．  
以下のコマンドでTFが取得できるか確認する．

```bash
ros2 run tf2_ros tf2_echo odom base_link
```

座標変換が表示されれば，`odom -> base_link` のTFは取得できている．

また，必要に応じてトピックを制限せずにrosbagを再生する．

```bash
cd ~/bags
ros2 bag play rosbag2_2026_01_31-16_40_55 --clock
```

> **画像を入れる場所**  
> ここに `tf2_echo odom base_link` の結果のスクリーンショットを貼る．  
> 例：`images/tf2_echo_odom_base_link.png`

```markdown
![odomからbase_linkへのTF確認](images/tf2_echo_odom_base_link.png)
```

---

## 6. RViz2で地図を確認

別ターミナルでRViz2を起動する．

```bash
rviz2
```

RViz2では以下のように設定する．

1. `Fixed Frame` を `map` に設定する
2. `Add` を選択する
3. `By Topic` から `/map` を追加する

地図が正常に生成されている場合，RViz2上にOccupancy Gridが表示される．

> **画像を入れる場所**  
> ここにRViz2で `/map` が表示されているスクリーンショットを貼る．  
> 例：`images/rviz_map_display.png`

```markdown
![RViz2でmapを表示](images/rviz_map_display.png)
```

---

## 7. `/map` の確認

RViz2だけでなく，コマンドからも `/map` が生成されているか確認する．

```bash
ros2 topic echo /map --once
```

今回の確認では，以下のように地図情報を取得できた．

```yaml
header:
  frame_id: map
info:
  resolution: 0.10000000149011612
  width: 359
  height: 421
  origin:
    position:
      x: -11.13503746005241
      y: -9.161342724147449
      z: 0.0
```

`width` と `height` が0ではないため，slam_toolboxによって地図が生成されていることを確認できた．

> **画像を入れる場所**  
> ここに `ros2 topic echo /map --once` の実行結果のスクリーンショットを貼る．  
> 例：`images/topic_echo_map.png`

```markdown
![mapトピックの確認](images/topic_echo_map.png)
```

---

## 8. 地図の保存

地図が生成されていることを確認したら，slam_toolboxを起動したまま地図を保存する．

```bash
mkdir -p ~/map
ros2 run nav2_map_server map_saver_cli -f ~/map/tudanuma_test_sakaitaisei_map
```

保存に成功すると，以下の2つのファイルが生成される．

```text
tudanuma_test_sakaitaisei_map.pgm
tudanuma_test_sakaitaisei_map.yaml
```

確認する．

```bash
ls ~/map
```

地図画像を開く場合は以下を実行する．

```bash
xdg-open ~/map/tudanuma_test_sakaitaisei_map.pgm
```

yamlファイルの中身を確認する場合は以下を実行する．

```bash
cat ~/map/tudanuma_test_sakaitaisei_map.yaml
```

> **画像を入れる場所**  
> ここに保存された `.pgm` 地図画像を貼る．  
> 例：`images/tudanuma_test_sakaitaisei_map.png`

```markdown
![作成した津田沼チャレンジ用地図](images/tudanuma_test_sakaitaisei_map.png)
```

---

## 9. 作成した地図の次工程での使い道

この工程で作成した地図は，次の工程で使用する．

| 作成物 | 次に使う場所 |
|---|---|
| `tudanuma_test_sakaitaisei_map.pgm` | Nav2のmap_server，GIMPでのkeepout地図作成 |
| `tudanuma_test_sakaitaisei_map.yaml` | Nav2のmap_server，オフライン自己位置推定 |
| `/map` の確認結果 | 地図が有効か判断する材料 |

作成した地図は，その後の以下の工程につながる．

1. オフライン自己位置推定  
   - 作成した地図が自己位置推定に使えるか確認する
2. GIMPによるkeepout地図作成  
   - 進入禁止領域や走行に不要な線を編集する
3. waypoint配置  
   - 作成した地図上に走行経路を設定する
4. box3への移行  
   - 地図ファイルを実機のNav2起動環境に配置する

---

## 10. トラブルシューティング

### `/map` が表示されない

RViz2で `/map` が表示されない場合は，以下を確認する．

```bash
ros2 topic list
```

`/map` が存在するか確認する．

```bash
ros2 topic echo /map --once
```

`width` と `height` が0の場合，地図が生成されていない可能性がある．

---

### `Failed to compute odom pose` が出る

slam_toolboxで以下の警告が出る場合がある．

```text
[slam_toolbox]: Failed to compute odom pose
```

この場合，以下を確認する．

```bash
ros2 run tf2_ros tf2_echo odom base_link
```

また，`slam_toolbox_params.yaml` の以下の設定を確認する．

```yaml
odom_frame: odom
base_frame: base_link
map_frame: map
scan_topic: /scan
use_sim_time: true
```

---

### RViz2で地図が見えにくい

RViz2上で地図が表示されているが見えにくい場合は，以下を試す．

- `Fixed Frame` を `map` にする
- マウスホイールでズームアウトする
- `Views` の `Zero` を押す
- MapのStatusを確認する

---

### rosbagのパスが見つからない

以下のようなエラーが出る場合がある．

```text
Bag path 'tudanuma_test_sakaitaisei' does not exist!
```

この場合，現在いるディレクトリにrosbagフォルダが存在しない．  
rosbagのある場所に移動してから再生する．

```bash
cd ~/bags
ls
```

今回使用したrosbagは以下である．

```text
rosbag2_2026_01_31-16_40_55
```

---

## 11. まとめ

本章では，box3で取得したrosbagを用いて，slam_toolboxにより2D地図を作成した．  
作成された `/map` を確認したところ，`width` と `height` が0ではなく，地図が生成されていることを確認できた．

また，`nav2_map_server` の `map_saver_cli` を用いて，Nav2で使用するための `.pgm` と `.yaml` を保存する．

作成した地図は，オフライン自己位置推定，keepout地図作成，waypoint配置，box3でのNav2走行に使用する．