# emcl2_ros2（自己位置推定）



## 概要



`emcl2_ros2`は、地図、LiDARのスキャンデータ、オドメトリを用いて、ロボットの自己位置を推定するROS 2パッケージです。



このページでは、屋外走行時に取得したrosbagを使用して、**オフラインで自己位置推定を実行し、RViz2で確認するまでの手順**を説明します。



パラメータの詳しい調整方法は、以下のページを参照してください。



- [オドメトリ誤差パラメータの調整](./emcl2_1.md)

- [膨張リセットとその他のパラメータの調整](./emcl2_2.md)



---



## 環境



- **OS**：Ubuntu 22.04 LTS

- **ROS 2**：Humble



---



## 1. 必要なパッケージの準備



### RViz2



RViz2がインストールされていない場合は、次のコマンドを実行します。



```bash

sudo apt install ros-humble-rviz2

```



### emcl2_ros2



ワークスペースの`src`ディレクトリへ移動し、`emcl2_ros2`を取得します。



```bash

cd ~/ros2_ws/src

git clone https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2.git

```



### その他の必要なパッケージ



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



新しい端末を開いた場合も、次のコマンドを実行してください。



```bash

source ~/ros2_ws/install/setup.bash

```



---



## 2. パラメータファイルの確認



今回使用するパラメータファイルは、次のファイルです。



- [emcl2.param.yaml](https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2/blob/main/config/emcl2.param.yaml)



### Box3で使用する場合



Box3のTF構成に合わせて、`footprint_frame_id`を変更します。



```yaml

# 変更前

footprint_frame_id: "base_footprint"



# 変更後

footprint_frame_id: "base_link"

```



> 

> この変更はBox3のTF構成に合わせるためのものです。  

> 別のロボットを使用する場合は、そのロボットのTF構成を確認して設定してください。



---



## 3. rosbagの中身を確認



使用するrosbagに必要なTopicが記録されているか確認します。



```bash

ros2 bag info rosbag2_2026_01_31-16_40_55

```



少なくとも、次のTopicが必要です。



- `/odometry/filtered`

- `/tf`

- `/tf_static`

- `/scan`



![rosbagの中身の確認](images/rosbag_info.png)



> [!NOTE]

> rosbagのファイル名は、使用するものに変更してください。



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



> 

> `.yaml`ファイルだけでなく、地図画像である`.pgm`ファイルも必要です。  

> また、`.yaml`内の`image`に記述されている画像ファイルのパスも確認してください。



---



## 5. 起動手順



端末を3つ使用します。



### 端末1：rosbagを再生



```bash

ros2 bag play rosbag2_2026_01_31-16_40_55 \

  --clock \

  --topics /odometry/filtered /tf /tf_static /scan

```



![rosbagの再生](images/rosbag_play.png)



再生直後に一度停止したい場合は、端末上で`Space`キーを押します。



パラメータ変更前後を比較する場合は、毎回同じrosbagを使用してください。



---



### 端末2：emcl2_ros2を起動



```bash

ros2 launch emcl2 emcl2.launch.py \

  map:=/home/rosuser/box3_ws/src/orne-box/orne_box_navigation_executor/config/maps/cit_3f_map.yaml \

  use_sim_time:=true

```



> [!NOTE]

> `map:=`の後ろは、使用する地図の`.yaml`ファイルのパスへ変更してください。



![emcl2_ros2の起動](images/emcl2_launch.png)



起動時には、端末に次のような情報が表示されます。



```text

ALPHA: 0.300000 / 0.500000

RESET

```



`RESET`は、膨張リセットが実行されたことを表します。



起動直後や初期位置が地図と合っていない場合には、リセットが発生することがあります。  

RViz2で初期位置を設定した後、パーティクルが正しい位置へ収束するか確認してください。



---



### 端末3：RViz2を起動



```bash

rviz2

```



---



## 6. RViz2の表示設定



### Fixed Frameの設定



RViz2左側の`Global Options`を開き、`Fixed Frame`を次のように設定します。



```text

map

```



### 表示するTopic



RViz2左下の`Add`を押し、`By topic`から次のTopicを追加します。



| Topic | 表示形式 | 確認する内容 |

|---|---|---|

| `/map` | Map | 事前に作成した地図 |

| `/scan` | LaserScan | LiDARのスキャンデータ |

| `/particlecloud` | PoseArray | パーティクルの分布 |

| `/mcl_pose` | PoseWithCovariance | 推定された自己位置 |



![RViz2のTopic追加](images/rviz_add_topics.png)



追加方法は次のとおりです。



1. RViz2左下の`Add`を押す

2. `By topic`を選択する

3. 追加するTopicを選ぶ

4. `OK`を押す

5. 必要なTopicを1つずつ追加する



---



## 7. 初期位置の設定



RViz2上部の`2D Pose Estimate`を選択します。



地図上でロボットの初期位置をクリックし、ドラッグして向きを設定します。



初期位置を設定した後、次の状態になるか確認してください。



- パーティクルが設定した位置の周辺に集まる

- LaserScanと地図の壁が重なる

- rosbagの再生に合わせて推定位置が移動する



rosbagを一時停止している場合は、端末1で再度`Space`キーを押して再生します。



---



## 8. 正常に動作しているかの確認



次の項目を確認します。



- ✅ 地図がRViz2に表示されている

- ✅ LaserScanと地図が大きくずれていない

- ✅ パーティクルがロボット周辺に収束している

- ✅ rosbagの再生に合わせて自己位置が移動している

- ✅ 一時的にパーティクルが広がっても、その後に再収束する

- ✅ 膨張リセットが繰り返され続けていない



今回使用した屋外走行rosbagでは、パラメータファイルの初期設定でも、明確な自己位置推定の破綻は確認されていません。



そのため、正常に動作している場合は無理にパラメータを変更せず、問題が確認された場合に調整を行います。



---



## 9. パラメータ調整へ進む



直進時や回転時に自己位置がずれる場合は、次のページを参照してください。



- [オドメトリ誤差パラメータの調整](./emcl2_1.md)



自己位置がずれた後に復帰しない場合や、膨張リセットを調整する場合は、次のページを参照してください。



- [膨張リセットとその他のパラメータの調整](./emcl2_2.md)



---



## トラブルシューティング



### 地図が表示されない



次の項目を確認してください。



- `map:=`に指定したパスが正しいか

- `.yaml`と`.pgm`の両方が存在するか

- `.yaml`内の`image`のパスが正しいか

- RViz2の`Fixed Frame`が`map`になっているか



### パーティクルが表示されない



次の項目を確認してください。



```bash

ros2 topic list

```



一覧に`/particlecloud`が存在するか確認します。



また、RViz2で`/particlecloud`が`PoseArray`として追加されているか確認してください。



### LaserScanが表示されない



次のコマンドで、`/scan`が配信されているか確認します。



```bash

ros2 topic echo /scan --once

```



### TFに関するエラーが出る



次の項目を確認してください。



- `base_link`と`odom`のTFが存在するか

- `footprint_frame_id`がロボットのTF構成と一致しているか

- rosbagに`/tf`と`/tf_static`が記録されているか



### rosbagを再生しても動かない



次の2つが両方設定されているか確認します。



rosbag再生側：



```bash

--clock

```



emcl2_ros2起動側：



```bash

use_sim_time:=true

```



---



## 参考リンク



- [emcl2_ros2](https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2)

- [emcl2_ros2のパラメータファイル](https://github.com/CIT-Autonomous-Robot-Lab/emcl2_ros2/blob/main/config/emcl2.param.yaml)

- [orne-boxの地図ファイル](https://github.com/open-rdc/orne-box/tree/humble-devel/orne_box_navigation_executor/config/maps)