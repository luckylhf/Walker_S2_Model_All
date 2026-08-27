# walker_s2_all

Walker S2 的 ROS 模型资源包，包名为 `walker_s2_all`。

## URDF 与网格

`urdf/` 提供两套机器人本体和四种末端执行器的组合，共 10 个 URDF（灵巧手有 v3 官方版 / v4 两代）：

| 本体 | 末端执行器 | 文件 |
| --- | --- | --- |
| simple | 夹板 | `s2_v1_simple_clamp.urdf` |
| simple | 无执行器 | `s2_v1_simple_noactuator.urdf` |
| simple | 灵巧手 v3 | `s2_v1_simple_hand_v3.urdf` |
| simple | 灵巧手 v4 | `s2_v1_simple_hand_v4.urdf` |
| simple | 夹爪 | `s2_v1_simple_gripper.urdf` |
| complex | 夹板 | `s2_v1_complex_clamp.urdf` |
| complex | 无执行器 | `s2_v1_complex_noactuator.urdf` |
| complex | 灵巧手 v3 | `s2_v1_complex_hand_v3.urdf` |
| complex | 灵巧手 v4 | `s2_v1_complex_hand_v4.urdf` |
| complex | 夹爪 | `s2_v1_complex_gripper.urdf` |

`meshes/` 中有与 URDF 对应的两套 STL：

- `s2_v1_simple/`：simple 本体网格。
- `s2_v1_complex/`：complex 本体网格。
- `ee_gripper/`：夹爪网格（simple/complex 共用）。
- `ee_hand_v3_simple/`、`ee_hand_v3_complex/`：三代灵巧手网格（对应simple/complex两种精度）。
- `ee_hand_v4_simple/`、`ee_hand_v4_complex/`：四代灵巧手网格（对应simple/complex两套精度）。
- `ee_clamp_simple/`、`ee_clamp_complex/`：夹板网格（对应simple/complex两种精度）。

### simple 与 complex 的差异

两者的运动学与动力学定义相同：对应 URDF 的链路数、关节拓扑、关节限位、质量和惯量均一致。差异只在本体 STL 网格精度：

| 本体 | 身体 STL 数量 | 三角面数量（约） | 网格文件大小（约） | 适用场景 |
| --- | ---: | ---: | ---: | --- |
| simple | 33 | 16 万 | 7.8 MB | RViz、快速加载、训练与日常调试 |
| complex | 33 | 285 万 | 136 MB | 高精度视觉展示与离线渲染 |

（上表仅统计身体网格 `s2_v1_simple/`、`s2_v1_complex/`；末端执行器网格位于 `ee_*` 目录，夹爪为两精度共用，灵巧手与夹板各有两精度。）

因此，选择 simple 或 complex 不会改变机器人质量、质心、惯量、关节自由度或末端执行器安装关系；只会影响视觉几何精度、显存占用和加载速度。

末端执行器均安装在左右 `L_sixforce_link` / `R_sixforce_link`。

### 质量口径

- 身体 33 个 link 的质量/惯量为原始 CAD 导出口径，未修改。
- 灵巧手 v3 / v4 的 26 个手部 link 质量与惯量已按官方规格**等比缩放并对齐**：v3 单手精确 559 g，v4 单手精确 815 g（官方手包 CAD 原始口径分别为 645.8 g / 489.3 g，与实物规格有偏差；分布比例保留 CAD 口径，质心位置不变）。

## MuJoCo

`mujoco/` 提供 simple 本体的 5 个 MJCF/XML：夹板、无执行器、灵巧手 v3、灵巧手 v4、夹爪。

- 夹爪为左右各一个位置控制输入，双指同步；控制范围为 0 到 -0.05 m，单侧力上限 140 N。
- 灵巧手（v3/v4）各使用 22 个手指关节，关节限位与力矩上限（±1.35 N·m）与对应 URDF 完全一致，单手质量分别为 559 g（v3）/ 815 g（v4），零位运动学与 URDF 逐位等价。

complex 网格超过 MuJoCo 的默认三角面限制，因此不提供 complex MJCF。

## USD（独立资产）

`USD/` 是一套**独立、自包含的 USD 资产包**，主入口为 `USD/s2_v1.usd`，其子层、材质与纹理位于 `USD/SubUSDs/`。

USD 与本包的 `urdf/`、`mujoco/*.xml`、`meshes/**/*.STL` **没有生成、引用、同步或参数继承关系**。它们是不同来源、不同版本链路的资产：

- 不要把 USD 的质量、惯量、关节数量、驱动参数或末端执行器配置当作 URDF/MJCF 的依据。
- 修改 URDF、MJCF 或 STL 后，不会自动更新 USD；修改 USD 也不会反向更新其它格式。
- 当前 USD 内容为独立的 `s2_hand4_v1` 双灵巧手资产，不等同于本包 10 个 URDF 或 5 个 MJCF 的任一版本。

USD 的材质资源已随 `USD/SubUSDs/materials/` 和 `USD/SubUSDs/textures/` 保存；其中 `OmniPBR.mdl` 由 Isaac Sim / Omniverse 运行环境提供。

## ROS 文件

- `launch/display.launch.py`：默认加载 `s2_v1_simple_clamp.urdf`。
- `config/rviz/`：RViz 显示配置。
- `package.xml`、`CMakeLists.txt`：ROS 2 包定义与安装规则。
