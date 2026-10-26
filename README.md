# LIO-SLAM

基于 Livox LiDAR 的激光—惯性里程计与建图工程。核心算法位于 `LIO-Livox`，使用单个 Livox LiDAR 及其内置 IMU 完成点云特征提取、运动畸变补偿、位姿估计和地图维护。

步兵底层控制（小陀螺算法）位于：https://github.com/WestaRlyh/Infantry_Gyroscope

## 演示视频

<table>
  <tr>
    <td align="center" width="100%">
      <strong>SLAM Mapping Demo</strong><br>
      <video src="https://github.com/user-attachments/assets/5e251fdf-0322-4bfa-b1c2-513cee2033f2" controls muted playsinline width="100%"></video>
    </td>
  </tr>
</table>


## 主要功能

- 基于 ROS 的 LiDAR—IMU 紧耦合里程计与建图。
- 从原始点云中提取角点、面点和不规则特征。
- 使用点云分割过滤动态目标，提升动态场景下的鲁棒性。
- 支持纯 LiDAR 里程计、IMU 预积分去畸变和 LiDAR—IMU 紧耦合模式。
- 支持 Livox Horizon、HAP 和 Mid-360 配置。
- 并行进行位姿估计与全局特征地图维护。

## 工程结构

```text
LIO_SLAM/
├── data/
├── LIO-Livox/
│   ├── config/                # Horizon、HAP、Mid-360 配置
│   ├── launch/                # ROS 启动文件
│   ├── include/               # 头文件
│   ├── src/                   # 算法源码
│   ├── rviz_cfg/              # RViz 配置
│   └── CMakeLists.txt
└── README.md
```

## 依赖环境

原工程在以下环境中测试：

- Ubuntu 16.04 / 18.04
- ROS Kinetic / Melodic
- Eigen3
- PCL
- Ceres Solver
- OpenCV
- SuiteSparse
- `livox_ros_driver`

安装 SuiteSparse：

```bash
sudo apt-get install libsuitesparse-dev
```

## 编译

将本仓库放入 catkin 工作空间的 `src` 目录后编译：

```bash
cd ~/catkin_ws
catkin_make
source devel/setup.bash
```

## 运行

### 使用 rosbag

先启动对应传感器配置：

```bash
roslaunch lio_livox horizon.launch
```

然后在另一个终端播放数据：

```bash
rosbag play YOUR_ROSBAG.bag
```

可用启动文件如下：

| 启动文件 | 传感器 | 配置文件 | IMU 模式 |
| --- | --- | --- | --- |
| `horizon.launch` | Livox Horizon | `horizon_config.yaml` | 紧耦合 |
| `hap.launch` | Livox HAP | `hap_config.yaml` | 不使用 IMU |
| `mid360.launch` | Livox Mid-360 | `mid360_config.yaml` | 紧耦合 |

### 使用实际设备

先启动 Livox 驱动：

```bash
roslaunch livox_ros_driver livox_lidar_msg.launch
```

再启动对应的 LIO-Livox 配置，例如：

```bash
roslaunch lio_livox mid360.launch
```

## ROS 话题

- 点云：`/livox/lidar`，消息类型为 `livox_ros_driver/CustomMsg`
- IMU：`/livox/imu`，消息类型为 `sensor_msgs/Imu`

## 关键参数

- `IMU_Mode=0`：不使用 IMU，使用恒速模型补偿运动畸变。
- `IMU_Mode=1`：使用 IMU 预积分补偿运动畸变。
- `IMU_Mode=2`：LiDAR 与 IMU 紧耦合估计。
- `Use_seg=1`：启用动态目标分割；静态场景可在对应 YAML 配置中关闭。
- `Extrinsic_Tlb`：LiDAR 与 IMU 之间的外参，需要根据传感器安装关系标定。

## 参考与致谢

本工程基于 Livox 的 LIO-Livox 实现，并参考了 LOAM、VINS-Mono、LIO-mapping、ORB-SLAM3、LiLi-OM 等工作。
