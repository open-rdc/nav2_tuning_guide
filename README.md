# nav2_tuning_guide
## 地図作成 (slam_toolbox)

[slam_toolbox](https://github.com/SteveMacenski/slam_toolbox) を使用して、Nav2の自律移動の基盤となる2D地図 (Occupancy Grid Map) を作成します。
ここでは、事前に取得したrosbagデータ（オフラインデータ）を用いて地図を生成する手順を説明します。
- [map_1.md](./map/map_1.md): 地図作成方法に関して
- [slam_toolbox.md](./map/slam_toolbox.md): slam_toolboxでの地図作成方法
- [glim.md](./map/glim.md): glimでの地図作成方法
- [tsukuba_map.md](./map/tsukuba_map.md): 大規模屋外環境での地図解像度設定
