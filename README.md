# walker_s2_all

Walker S2 的 ROS 模型资源包，包名为 `walker_s2_all`。

## URDF 与网格

`urdf/` 提供两套机器人本体和四种末端执行器的组合，共 8 个 URDF：

| 本体 | 末端执行器 | 文件 |
| --- | --- | --- |
| simple | 夹板 | `s2_v1_simple_clamp.urdf` |
| simple | 无执行器 | `s2_v1_simple_noactuator.urdf` |
| simple | 灵巧手 | `s2_v1_simple_hand.urdf` |
| simple | 夹爪 | `s2_v1_simple_gripper.urdf` |
| complex | 夹板 | `s2_v1_complex_clamp.urdf` |
| complex | 无执行器 | `s2_v1_complex_noactuator.urdf` |
| complex | 灵巧手 | `s2_v1_complex_hand.urdf` |
| complex | 夹爪 | `s2_v1_complex_gripper.urdf` |

`meshes/` 中有与 URDF 对应的两套 STL：

- `s2_v1_simple/`：simple 本体网格。
- `s2_v1_complex/`：complex 本体网格。

### simple 与 complex 的差异

两者的运动学与动力学定义相同：对应 URDF 的链路数、关节拓扑、关节限位、质量和惯量均一致。差异只在本体 STL 网格精度：

| 本体 | STL 数量 | 三角面数量（约） | 网格文件大小（约） | 适用场景 |
| --- | ---: | ---: | ---: | --- |
| simple | 67 | 83 万 | 42 MB | RViz、快速加载、训练与日常调试 |
| complex | 67 | 352 万 | 176 MB | 高精度视觉展示与离线渲染 |

因此，选择 simple 或 complex 不会改变机器人质量、质心、惯量、关节自由度或末端执行器安装关系；只会影响视觉几何精度、显存占用和加载速度。

末端执行器均安装在左右 `L_sixforce_link` / `R_sixforce_link`。

## MuJoCo

`mujoco/` 提供 simple 本体的 4 个 MJCF/XML：夹板、无执行器、灵巧手、夹爪。

- 夹爪为左右各一个位置控制输入，双指同步；控制范围为 0 到 -0.05 m，单侧力上限 140 N。
- 灵巧手使用 22 个手指关节；其力矩上限按四指指尖力 7 N、拇指指尖力 6 N、五指总握力 25 N 的规格设置。

complex 网格超过 MuJoCo 的默认三角面限制，因此不提供 complex MJCF。

## USD（独立资产）

`USD/` 是一套**独立、自包含的 USD 资产包**，主入口为 `USD/s2_v1.usd`，其子层、材质与纹理位于 `USD/SubUSDs/`。

USD 与本包的 `urdf/`、`mujoco/*.xml`、`meshes/**/*.STL` **没有生成、引用、同步或参数继承关系**。它们是不同来源、不同版本链路的资产：

- 不要把 USD 的质量、惯量、关节数量、驱动参数或末端执行器配置当作 URDF/MJCF 的依据。
- 修改 URDF、MJCF 或 STL 后，不会自动更新 USD；修改 USD 也不会反向更新其它格式。
- 当前 USD 内容为独立的 `s2_hand4_v1` 双灵巧手资产，不等同于本包 8 个 URDF 或 4 个 MJCF 的任一版本。

USD 的材质资源已随 `USD/SubUSDs/materials/` 和 `USD/SubUSDs/textures/` 保存；其中 `OmniPBR.mdl` 由 Isaac Sim / Omniverse 运行环境提供。

## ROS 文件

- `launch/display.launch.py`：默认加载 `s2_v1_simple_clamp.urdf`。
- `config/rviz/`：RViz 显示配置。
- `package.xml`、`CMakeLists.txt`：ROS 2 包定义与安装规则。
