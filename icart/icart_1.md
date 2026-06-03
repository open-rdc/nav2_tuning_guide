# オドメトリの調整方法

以下の説明は [ROS WIKI Navigation Tuning Guide](https://wiki.ros.org/navigation/Tutorials/Navigation%20Tuning%20Guide#Odometry) を参考にしています。何よりもまず初めにオドメトリを調整することをおすすめします。調整するパラメータは以下の2つです。

- RADIUS（タイヤ半径）
- TREAD（トレッド幅：左右の車輪間距離）
  - box3では `orne_box_bringup/config/ypspur/box_v3.param` で設定されています

---

## 1. 調整の準備

まず、調整のための準備を行います。

注意: センサフュージョンや自己位置推定（AMCL/EKF等）が動いている場合はオドメトリのみの出力にしてください。普段の `orne_box_bringup.launch.py` や `play_waypoints_nav.launch.py` を立ち上げてしまうと、自動補正がかかり純粋なタイヤのズレを確認できなくなります。キャリブレーション時は必ず以下の手順で立ち上げてください。

### 起動手順（端末を3つ使用）

1. 足回り＋LiDARのみの起動（EKF等の補正なし）

```bash
ros2 launch orne_box_bringup orne_box_drive.launch.py
```

2. 手動操作ノードの起動（コントローラーがある場合は不要）

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

3. RViz2の起動

```bash
rviz2
```

RViz2が開いたら、Global Options の Fixed Frame を `odom` に設定します。そして add パネルから `LaserScan` を追加し、その Decay Time を高く(20〜30くらい)設定します。LaserScan の topic も指定してください。(`/scan` 等)

設定を save させておくと次回以降楽です。

## 2. 並進成分のテスト

準備が整ったら、並進成分のテストを行います。壁から数メートル離してロボットを起動させ、先程設定した RViz を立ち上げます。壁に向かって 1 メートル走らせ、スキャンに数センチ以上の厚みがないことを確認します。数メートルに渡ってスキャンが前後に広がっていたら `box_v3.param` の `RADIUS` を調整してください。調整後は必ず 1 のコマンド (`orne_box_drive.launch.py`) を再起動して設定を反映させます。

## 3. 回転成分のテスト

次に回転成分のテストを行います。先程同様、設定した RViz を立ち上げ、ロボットをその場で回転 (360度) させます。理想はスキャンがぴったり重なることですが、多少の回転ドリフトは予想されるので、スキャンが 1～2 度以上ずれていないことを確認します。それよりも景色が大きくずれている場合、`box_v3.param` の `TREAD` を調整してください。

最後に、このテストを終えたら、もう一度 2. 並進成分のテストを実行してください。並進に悪影響が出ておらず、ずれがないことを再確認してオドメトリの調整は完了です。
