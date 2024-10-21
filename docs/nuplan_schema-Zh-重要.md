nuPlan Schema
==========

This document describes the database schema used in nuPlan. All annotations and metadata (including calibration, maps, vehicle coordinates etc.) are covered in relational databases. The database tables are listed below. Every row can be identified by its unique primary key `token`. Foreign keys such as `log_token` are used to link to the `token` of the table `log`.    本文档描述了nuPlan中使用的数据集架构。所有注释和元数据（包括标定、地图、车辆坐标等）都包含在relational数据集中【.db文件】。下面列出了数据集tables。每一行都可以通过其唯一的主键“token”来识别。诸如“log_token”之类的外键用于链接到表“log”的“token”。

> nuPlan schema的说明：
>
> - 每一个table都有一个唯一的标识符叫做`token`，table中可能会有外部键keys 表示为`<table>_token`。
> - 我们只展示了存储载DB文件的字段fields，而没有展示可以通过函数方便索引的一些附加信息。
> - Map对象的标识符表示为`<map_layer>_id(s)`

![](nuplan_schema.png)

log
---------
Information about the log from which the data was extracted.关于从中提取数据的日志的信息。
```json
log {
   "token":                        <str> -- Unique record identifier.
   "vehicle_name":                 <str> -- Vehicle name.  车辆名称
   "date":                         <str> -- Date (YYYY-MM-DD).
   "timestamp":                    <int> -- Unix timestamp for when the log started.
   "logfile":                      <str> -- Original log file name.
   "location":                     <str> -- Area where log was captured, e.g. Singapore. 捕获log时的地区，如：las_vegas
   "map_version":                  <str> -- Name of map version used in this log.  地图版本名称，如：us-nv-las-vegas-strip
}
```


ego_pose
---------
Ego vehicle pose at a particular timestamp.  特定时间戳的自车位姿
```
ego_pose {
   "token":                        <str> -- Unique record identifier.
   "log_token":                    <str> -- Foreign key. Identifies the log which this ego pose is a part of.
   "timestamp":                    <str> -- Unix timestamp.
   "x":                            <float> -- Ego vehicle location center_x in global coordinates(in meters).
   "y":                            <float> -- Ego vehicle location center_y in global coordinates(in meters).
   "z":                            <float> -- Ego vehicle location center_z in global coordinates(in meters).
   "qw":                           <float> -- Ego vehicle orientation in quaternions in global coordinates.
   "qx":                           <float> -- Ego vehicle orientation in quaternions in global coordinates.
   "qy":                           <float> -- Ego vehicle orientation in quaternions in global coordinates.
   "qz":                           <float> -- Ego vehicle orientation in quaternions in global coordinates.
   "vx":                           <float> -- Ego vehicle velocity x in local coordinates(in m/s).
   "vy":                           <float> -- Ego vehicle velocity y in local coordinates(in m/s).
   "vz":                           <float> -- Ego vehicle velocity z in local coordinate(in m/s)s.
   "acceleration_x":               <float> -- Ego vehicle acceleration x in local coordinates(in m/s2).
   "acceleration_y":               <float> -- Ego vehicle acceleration y in local coordinates(in m/s2).
   "acceleration_z":               <float> -- Ego vehicle acceleration z in local coordinates(in m/s2).
   "angular_rate_x":               <float> -- Ego vehicle angular rate x in local coordinates.
   "angular_rate_y":               <float> -- Ego vehicle angular rate y in local coordinates.
   "angular_rate_z":               <float> -- Ego vehicle angular rate z in local coordinates.
   "epsg":                         <int> -- Ego vehicle epsg. Epsg is a latitude/longitude coordinate system based on the Earth's center of mass.
}
```


camera
---------
The `camera` table contains information about the calibration and other settings of a particular camera in a particular log.   “camera”表包含了关于在特定日志中特定相机的校准和其他设置的信息。
```
camera {
   "token":                        <str> -- Unique record identifier.——唯一记录标识符。
   "log_token":                    <str> -- Foreign key. Identifies the log that uses the configuration specified here.——外键。标识使用此处指定的配置的日志。
   "channel":                      <str> -- The camera name, which describes it's position on the car (e.g. 'CAM_F0', 'CAM_R0', 'CAM_R1', 'CAM_R2', 'CAM_B0', 'CAM_L0', 'CAM_L1', 'CAM_L2'). —相机名称，用于描述相机在汽车上的位置(例如:“CAM_F0”、
   "model":                        <str> -- The camera model used.—使用的相机型号。
   "translation":                  <float> [3] -- The extrinsic translation of the camera relative to the ego vehicle coordinate frame. Coordinate system origin in meters: x, y, z. ——相机相对于自我车辆坐标框架的外部平移。坐标系原点以米为单位:x y z。
   "rotation":                     <float> [4] -- The extrinsic rotation of the camera relative to the ego vehicle coordinate frame. Coordinate system orientation as quaternion: w, x, y, z.——相机相对于自我车辆坐标框架的外部旋转。坐标系方向为四元数:w x y z。
   "intrinsic":                    <float> [3, 3] -- Intrinsic camera calibration matrix.——内置摄像机校准矩阵。
   "distortion":                   <float> [*] -- The camera distortion parameters according to the Caltech model (k1, k2, p1, p2, k3)——根据Caltech模型的相机畸变参数(k1, k2, p1, p2, k3)
   "width":                        <int> -- The width of the camera image in pixels.——相机图像的宽度，单位为像素。
   "height":                       <int> -- The height of the camera image in pixels.           
}
```


image
---------
The `image` table stores metadata to retrieve a single image taken from a camera.  It does not store the image itself. ' image '表存储元数据以检索从相机拍摄的单张图像。它不存储图像本身。
```
image {
   "token":                        <str> -- Unique record identifier.
   "next_token":                   <str> -- Foreign key. Record that follows this in time. Empty if last image of log.
   "prev_token":                   <str> -- Foreign key. Record that precedes this in time. Empty if first image of log.
   "ego_pose_token":               <str> -- Foreign key. Indicates the ego pose at the time that the image was captured.
   "camera_token":                 <str> -- Foreign key. Indicates the camera settings used to capture the image.
   "filename_jpg":                 <str> -- Relative path to image file.
   "timestamp":                    <int> -- Unix timestamp.
}
```


lidar
---------
The `lidar` table contains information about the calibration and other settings of a particular lidar in a particular log.  “lidar”表包含关于特定日志中特定激光雷达的校准和其他设置的信息。
```
lidar {
   "token":                        <str> -- Unique record identifier.
   "log_token":                    <str> -- Foreign key. Identifies the log that uses the configuration specified here.
   "channel":                      <str> -- Log channel name.
   "model":                        <str> -- The lidar model.
   "translation":                  <float> [3] -- The extrinsic translation of the lidar relative to the ego vehicle coordinate frame. Coordinate system origin in meters: x, y, z.
   "rotation":                     <float> [4] -- The extrinsic rotation of the lidar relative to the ego vehicle coordinate frame. Coordinate system orientation as quaternion: w, x, y, z.
}
```


lidar_pc
---------
The `lidar_pc` table stores metadata to retrieve a single pointcloud taken from a lidar.  It does not store the pointcloud itself.
The pointcloud combines sweeps from multiple different lidars that are aggregated on the car.     

' lidar_pc '表存储元数据，用于检索从激光雷达获取的单个点云。它不存储点云本身。点云结合了聚集在汽车上的多个不同激光雷达的扫描结果。

```
lidar_pc {
   "token":                        <str> -- Unique record identifier.
   "next_token":                   <str> -- Foreign key. Record that follows this in time. Empty if end of log.——外键。在这个token之后的记录。if为空，表示日志结束。
   "prev_token":                   <str> -- Foreign key. Record that precedes this in time. Empty if start of log.——外键。在这个token之前的记录。如果日志开始为空。
   "scene_token":                  <str> -- Foreign key. References the scene that contains this lidar_pc.——外键。引用包含这个lidar_pc的场景。
   "ego_pose_token":               <str> -- Foreign key. Indicates the ego pose at the time that the pointcloud was captured.——外键。指示点云被捕获时的自车姿态。
   "lidar_token":                  <str> -- Foreign key. Indicates the lidar settings used to capture the pointcloud.——外键。指示用于捕获点云的激光雷达设置。
   "filename":                     <str> -- Relative path to pointcloud blob.
   "timestamp":                    <int> -- Unix timestamp.
}
```


lidar_box
---------
The `lidar_box` table stores individual object annotations in the form of 3d bounding boxes.        `lidar_box` 表以3d包围框的形式存储单个对象注释。
These boxes are extracted from an Offline Perception system and tracked across time using the `track` table.      这些盒子是从离线感知系统中提取出来的，并使用“`track`表跨时间跟踪。
Since the perception system uses lidar as an input modality, all `lidar_boxes` are linked to the `lidar_pc` table.  由于感知系统使用激光雷达作为输入方式，所有的`lidar_boxes` 都链接到`lidar_pc` 表。 

```
lidar_box {
   "token":                        <str> -- Unique record identifier.
   "lidar_pc_token":               <str> -- Foreign key. References the lidar_pc where this object box was detected.
   "track_token":                  <str> -- Foreign key. References the object track that this box was associated with.
   "next_token":                   <str> -- Foreign key. Record that follows this in time. Empty if end of track.
   "prev_token":                   <str> -- Foreign key. Record that precedes this in time. Empty if start of track.
   "x":                            <float> -- Bounding box location center_x in global coordinates(in meters).
   "y":                            <float> -- Bounding box location center_y in global coordinates(in meters).
   "z":                            <float> -- Bounding box location center_z in global coordinates(in meters).
   "width":                        <float> -- Bounding box size width(in meters).
   "length":                       <float> -- Bounding box size length(in meters).
   "height":                       <float> -- Bounding box size height(in meters).
   "vx":                           <float> -- Bounding box velocity v_x(in m/s). This quantity is estimated by a Neural Network and may be inconsistent with future positions.
   "vy":                           <float> -- Bounding box velocity v_y(in m/s). This quantity is estimated by a Neural Network and may be inconsistent with future positions.
   "vz":                           <float> -- Bounding box velocity v_z(in m/s). This quantity is estimated by a Neural Network and may be inconsistent with future positions.
   "yaw":                          <float> -- Bounding box orientation yaw.
   "confidence":                   <float> -- Bounding box confidence.
}
```


track
---------
An object track, e.g. particular vehicle. This table is an enumeration of all unique object instances we observed.  物体轨迹，例如特定的车辆。该表是我们观察到的所有唯一对象实例的枚举。
```
track {
   "token":                        <str> -- Unique record identifier.
   "category_token":               <str> -- Foreign key. Object instance category.——外键。对象实例类别。
   "width":                        <float> -- Bounding box size width(in meters).——边界框大小的宽度(以米为单位)。
   "length":                       <float> -- Bounding box size length(in meters).——3d包围框大小的长度(单位:米)。
   "height":                       <float> -- Bounding box size height(in meters).——边界框大小的高度(以米为单位)。
}
```


category
---------
Taxonomy of object categories (e.g. 'vehicle', 'bicycle', 'pedestrian', 'traffic_cone', 'barrier', 'czone_sign', 'generic_object').  对象类别的分类

> traffic_cone：锥形交通路标

```
category {
   "token":                        <str> -- Unique record identifier.
   "name":                         <str> -- Category name.
   "description":                  <str> -- Category description.
}
```


scene
---------
A `scene` is a snippet of upto 20s duration from a `log`.   一个`scene`是一个`log`中长达20秒的片段。
Every scene stores a goal for the ego vehicle that is a future ego pose from beyond that scene.In addition we provide a sequence of road blocks to navigate towards the goal.  每个场景都为自车存储了一个目标，即来自该场景之外的未来自车pose。此外，我们还提供了一系列通向目标的道路块road blocks。 

```json
scene {
   "token":                        <str> -- Unique record identifier.
   "log_token":                    <str> -- Foreign key. The log that this scene is a part of.——外键。这个场景是日志的一部分。
   "name":                         <str> -- Unique Scene Name.
   "goal_ego_pose_token":          <str> -- Foreign key. A future ego pose that serves as the goal for this scene.——外键。一个未来的自我姿态，作为这个场景的目标。
   "roadblock_ids":                <str> -- A sequence of roadblock ids separated by commas. The ids can be looked up in the Map API.——以逗号分隔的道路块id序列。这些id可以在Map API中查找。
}
```


scenario_tag
---------
An instance of a scenario extracted from the database. Scenarios are linked to `lidar_pcs` and represent the point in time when a scenario miner was triggered,   e.g. when simultaneously being in two lanes for `CHANGING_LANE`. Some scenario types optionally can refer to an agent that the ego is interacting with (e.g. in `STOPPING_WITH_LEAD`). 

从数据库中提取的场景实例。场景链接到 `lidar_pcs `，表示场景矿工被触发的时间点， 例如，当同时在两条车道 `CHANGING_LANE`时。某些场景类型选择性引用
与自车交互的其它动态智能体 (例如在`STOPPING_WITH_LEAD`中)。

```json
scenario_tag {
   "token":                        <str> -- Unique record identifier.
   "lidar_pc_token":               <str> -- Foreign key. The lidar_pc at which this scenario was triggered.  —外键。触发此场景的lidar_pc。
   "type":                         <str> -- Type of Scenario. Ex on_intersection, starting_unprotected_cross_turn etc. There are around 70 of these scenario types. ——场景类型。Ex on_intersection, starting_unprotected_cross_turn等。这种场景类型大约有70种。### 教程中可选的场景就是按照这个打的标签
   "agent_track_token":            <str> -- Foreign key. Token of the agent interacting with the ego vehicle. Can be none if there is no interacting agent.  —外键。代理与自车交互的标识符。如果没有相互作用的代理，则可以为none。
}
```


traffic_light_status
---------
We use the observed motion of agents in the environment to estimate the status of a traffic light. For simplicity the status is stored in a particular lane connector (an idealized path across an intersection), rather than in the traffic light itself.  我们利用环境中观察到的智能体运动来估计红绿灯的状态。为了简单起见，状态存储在特定的车道连接器(穿过十字路口的理想路径)中，而不是在红绿灯本身中。

```json
traffic_light_status {
   "token":                        <str> -- Unique record identifier.
   "lidar_pc_token":               <str> -- Foreign key. The traffic light status is based on the motion of the agents detected in this lidar_pc.
   "lane_connector_id":            <int> -- ID of lane connector in the map.
   "status":                       <str> -- Status of traffic light. Can be green, red or unknown.
}
```