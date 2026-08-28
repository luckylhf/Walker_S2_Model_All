# walker_s2_all

Walker S2 的 ROS 模型资源包，包名为 `walker_s2_all`。

除 Git 内部数据和 macOS 自动生成的 `.DS_Store` 外，当前包内共 255 个有效文件：10 个 URDF、5 个 MJCF、181 个 STL、53 个 USD 资产文件，以及 6 个 ROS/文档文件。

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

### 真机兼容基准

全部 10 个 URDF 的机器人本体均以真机验证通过的 `walker_s2_description/s2_v1.urdf` 为基准：

- 30 个本体活动关节的父子关系、局部坐标系、旋转轴、限位、力矩、速度、质量和惯量均与真机模型一致。
- 保留真机依赖的 `torso_link`、`head_base_link`、`L_arm_base_link`、`R_arm_base_link` 固定坐标系。
- 除夹爪外均保留末端安装所必需的 `L_sixforce_link` / `R_sixforce_link`，并重新计算它们与 GOOD 腕部坐标系之间的固定变换。
- 夹板、灵巧手 v3/v4 保留各自原有末端结构、关节和质量口径，并安装到对应 `sixforce_link`；夹爪采用真机模型分支，直接连接 `wrist_roll_link` 并提供 `L_tool_link` / `R_tool_link`。
- 本体网格通过逐 link 固定变换对齐新坐标系，因此替换运动学后仍保持原模型的零位外观。

改造后的规模如下：

| 末端类型 | link 数 | joint 数 |
| --- | ---: | ---: |
| 无执行器 | 37 | 36 |
| 夹板 | 39 | 38 |
| 夹爪 | 43 | 42 |
| 灵巧手 v3 / v4 | 63 | 62 |

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

夹板和灵巧手通过左右 `L_sixforce_link` / `R_sixforce_link` 安装；夹爪按真机模型直接安装到 `L_wrist_roll_link` / `R_wrist_roll_link`。

### 质量口径

- 本体质量、质心和惯量采用真机模型口径；计入左右 sixforce 接口后，无执行器模型总质量为 70.541408 kg。
- 夹爪按产品手册校正为单侧 1 kg：`gripper_base_link` 0.82 kg、`tool_link` 0.02 kg、两个手指 link 各0.08 kg。双侧夹爪共2 kg，对应机器人总质量为 72.050588 kg。
- 灵巧手 v3 / v4 的 26 个手部 link 质量与惯量已按官方规格**等比缩放并对齐**：v3 单手精确 559 g，v4 单手精确 815 g（官方手包 CAD 原始口径分别为 645.8 g / 489.3 g，与实物规格有偏差；分布比例保留 CAD 口径，质心位置不变）。

## MuJoCo

`mujoco/` 提供 simple 本体的 5 个 MJCF/XML：夹板、无执行器、灵巧手 v3、灵巧手 v4、夹爪。

- 五个 MJCF 的本体结构、关节坐标系、惯量和碰撞体与对应的新 URDF 一致，末端执行器和原有 actuator 数量保持不变。
- 夹爪为左右各一个位置控制输入，双指同步；按真机夹爪 ±0.0215 m 单指行程，控制范围为 0 到 -0.043 m，单侧力上限 140 N，单侧总质量 1 kg。
- 灵巧手（v3/v4）各使用 22 个手指关节，关节限位与力矩上限（±1.35 N·m）与对应 URDF 完全一致，单手质量分别为 559 g（v3）/ 815 g（v4），零位运动学与 URDF 逐位等价。

complex 网格超过 MuJoCo 的默认三角面限制，因此不提供 complex MJCF。

## USD（独立资产）

`USD/` 是一套**独立、自包含的 USD 资产包**，主入口为 `USD/s2_v1.usd`，其子层、材质与纹理位于 `USD/SubUSDs/`。

USD 与本包的 `urdf/`、`mujoco/*.xml`、`meshes/**/*.STL` **没有生成、引用、同步或参数继承关系**。它们是不同来源、不同版本链路的资产：

- 不要把 USD 的质量、惯量、关节数量、驱动参数或末端执行器配置当作 URDF/MJCF 的依据。
- 修改 URDF、MJCF 或 STL 后，不会自动更新 USD；修改 USD 也不会反向更新其它格式。
- 当前 USD 内容为独立的 `s2_hand4_v1` 双灵巧手资产，不等同于本包 10 个 URDF 或 5 个 MJCF 的任一版本。

USD 的材质资源已随 `USD/SubUSDs/materials/` 和 `USD/SubUSDs/textures/` 保存；其中 `OmniPBR.mdl` 由 Isaac Sim / Omniverse 运行环境提供。

当前 `CMakeLists.txt` 的 ROS 2 安装规则不包含 `USD/`；`colcon build` 后的 install space 中不会出现这套独立 USD 资产，需要时应直接使用源码目录中的 `USD/s2_v1.usd`。

## ROS 文件

- `launch/display.launch.py`：默认加载 `s2_v1_simple_clamp.urdf`。
- `config/rviz/`：RViz 显示配置。
- `package.xml`、`CMakeLists.txt`：ROS 2 包定义与安装规则。

`package.xml` 当前声明 Apache License 2.0，但源码目录中没有独立 `LICENSE` 文件。如果要对外发布该包，需由资产所有者确认版权归属并补入正确的许可文本。

## 自洽性校验

2026-08-28 对整个目录重新执行了以下校验：

- 10 个 URDF 均可解析，link/joint 名称无重复，树结构单根、无环且全连通，所有关节端点和网格路径都存在。
- 5 组 simple/complex URDF 的 link、joint、惯性、关节坐标系、轴、限位和 mimic 定义逐项相同；差异仅为网格及其视觉/碰撞对齐参数。
- 181 个 STL 均通过格式与非空检查，并且全部被至少一个 URDF 或 MJCF 引用；不存在缺失引用或孤立 STL。字节相同的左/右件仍有独立语义路径并被模型引用，不能按重复文件删除。
- 5 个 MJCF 均通过 MuJoCo 3.3.7 编译；在零位和两组限位内位姿下，全部同名 link 与对应 URDF 的最大前向运动学误差为 `4.446e-12`。
- ROS launch 文件通过 Python 语法检查；RViz 的 Fixed Frame 为所有 URDF 共同根 link `base_link`，机器人描述话题为 `/robot_description`。
- USD 的 10 个 layer、12 个 MDL 和 30 个纹理组成完整的内部引用链；`USD/.collect.mapping.json` 是收集来源与哈希记录，不参与运行时加载，但保留它有利于资产溯源。

夹爪质量修正后，tool 和手指保留原质心与惯性方向，惯量按质量比例缩放。base STL 不是闭合网格，因此 base 质心和惯量按其闭合凸包、0.82 kg 和均匀密度估算；该估算值已同步到 URDF 和 MJCF。

USD 不属于本次改造范围。通用 `usdchecker` 不能在当前非 Omniverse 环境中给出全通过结果：它无法解析由 Isaac Sim / Omniverse 提供的 `OmniPBR.mdl`，并会报告该资产使用的 Omniverse schema/材质校验差异。这不影响 URDF、MJCF 或 STL 的上述校验结果。

## 不需要的文件

以“模型、ROS 包、USD 资产或溯源文档是否使用”为判定口径，当前可确定不需要的包内文件只有以下 8 个 macOS 缓存文件：

- `.DS_Store`
- `USD/.DS_Store`
- `USD/SubUSDs/.DS_Store`
- `config/.DS_Store`
- `meshes/.DS_Store`
- `meshes/s2_v1_complex/.DS_Store`
- `meshes/s2_v1_simple/.DS_Store`
- `urdf/.DS_Store`

本次只列出、不删除。`.gitignore` 已包含 `.DS_Store`，它们不会进入版本控制。除上述文件外，未发现可以在不破坏当前 URDF/MJCF/USD/ROS 用途的前提下直接删除的文件。
