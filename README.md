<div align="center">

# Hephaestus

</div>

> AgroTech 协会中型轮式机器人平台（仓库：`Competition-Robot-2026-Hephaestus`）

> 本仓库由原集合仓库 `Steering-Wheel-Chassis`（一车一目录）拆分而来，当前只维护 **Hephaestus** 这一台机器人

---

## 1. 当前状态

- **状态：** 开发中（智械争锋全自主区主线）
- **开发计划：** [`docs/plan.md`](docs/plan.md)
- **整车详细说明：** [`Hephaestus/README.md`](Hephaestus/README.md)

> 首次形成可复现的稳定版本后，再创建 Git Tag + GitHub Release，并将本 README 更新为该稳定版本的完整使用说明

---

## 2. 参与本项目开发

推荐流程：

**Issue → Branch → Commit → Push → Pull Request → 项目负责人 Merge**

- 开始开发前，原则上先创建或认领 Issue
- 请勿直接在 `main` 开发或 Push
- 如果已经误在 `main` 上产生了有用 Commit，**不要先 `reset --hard`**，先按协作指南把提交保存到新分支

完整流程与常见问题：[`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md)

---

## 3. 项目简介

`Hephaestus` 是 AgroTech 协会的中型轮式机器人平台，采用舵轮底盘，使用 STM32 控制板完成底层实时控制，树莓派 5 运行 ROS2 Humble 自主任务系统，并配有 PC 主臂遥操作与 ASRPro 语音链路。

当前主线是**智械争锋全自主区**：MCU 侧在安全条件满足后进入 AutoPi，Pi 端运行全自主运输状态机，依次完成语音启动、导航至智能分拣区、视觉识别分类标识、导航至待派送区、按分类结果派送至对应园区并上报 DONE/FAIL。

整车角色分工：

| 端 | 目录 | 作用 |
|---|---|---|
| MCU | `Hephaestus/chassis_control_code/` | 底盘实时控制、机械臂执行、AutoPi 状态机、安全边界、PC/Pi/ASR 通信 |
| Pi | `Hephaestus/chassis-pi-ws/` | ROS2 Humble、MCU bridge、Nav2 导航、视觉识别、全自主运输状态机 |
| PC | `Hephaestus/chassis-pc-ws/` | 主臂遥操作和上位机调试 |
| ASRPro | `Hephaestus/hephaestus_asrpro/` | 语音触发和播报链路 |

> 说明：本仓库由原集合仓库拆分而来，仓库内部的部分功能包与文档仍沿用 `atlas_*` 前缀命名（如 `atlas_autonomous_task`、`atlas_mission_manager`），属拆分迁移的遗留命名，后续可逐步统一；协作者按当前实际命名引用即可。

---

## 4. 环境要求

### 软件

- 树莓派 5，Ubuntu 22.04，ROS2 Humble
- 需安装：`navigation2`、`nav2-bringup`、`cartographer`、`cartographer-ros`、`xacro`、`robot-state-publisher`、`rviz2` 等，以及工作区 `requirements.txt` 管理的 Python 依赖
- PC 端遥操作 App 需要 `python3` + `requirements.txt` 依赖
- MCU 固件使用 STM32CubeMX / EIDE 工具链编译

### 硬件

- 主控：STM32 控制板（MCU 端）
- 上位机：树莓派 5（Pi 端）
- 传感器：LSLIDAR 雷达、IMU
- 执行器：舵轮底盘、五自由度机械臂、末端吸盘

---

## 5. 快速开始

### 5.1 MCU 侧

烧录 `Hephaestus/chassis_control_code/` 固件后，确认 Pi 能打开 MCU 串口，且能收到状态、里程计、IMU 和机械臂状态帧；MCU 仅在安全条件满足时接受 ASRPro 语音门控进入 AutoPi。

### 5.2 PC 遥操作

```bash
cd Hephaestus/chassis-pc-ws
python3 -m pip install -r requirements.txt
python3 scripts/main.py
```

### 5.3 Pi 端

```bash
cd Hephaestus/chassis-pi-ws
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash

ros2 launch robot_startup robot_start.launch.py
```

完整比赛流程（遥操作完成 → MCU AutoPi → ASRPro 播报 → 语音启动 → 全自主派送）与单独测试导航 / 视觉 / ASRPro 的命令，见 [`Hephaestus/docs/使用说明与教程.md`](Hephaestus/docs/使用说明与教程.md) 与 [`Hephaestus/docs/配置指南.md`](Hephaestus/docs/配置指南.md)。

---

## 6. 目录结构

```text
Competition-Robot-2026-Hephaestus/
├── Hephaestus/
│   ├── chassis_control_code/          # MCU 底盘与机械臂控制工程
│   ├── chassis-pc-ws/                 # PC 主臂遥操作 App
│   ├── chassis-pi-ws/                 # Pi 端 ROS2 自主任务工作区
│   ├── hephaestus_asrpro/             # ASRPro 语音链路工程
│   ├── docs/                          # 配置、使用、状态机、协议等说明文档
│   └── README.md                      # 整车说明（随拆分迁移更新中）
├── docs/
│   └── plan.md                        # 开发计划
└── README.md
```

---

## 7. 文档

- `Hephaestus/docs/配置指南.md`：启动入口、导航与视觉配置
- `Hephaestus/docs/使用说明与教程.md`：编译、一键启动、比赛流程、单模块测试
- `Hephaestus/docs/工作区详细介绍及完整任务流.md`：工作区结构与完整任务链路
- `Hephaestus/docs/状态机详细说明及错误处理.md`、`智械争锋全自主区状态机.md`：全自主运输状态机
- `Hephaestus/docs/comms_protocol.md`：PC / Pi / MCU 统一通信协议
- `Hephaestus/docs/ASRPRO_TWEN51_通信协议.md`、`串口配置说明.md`、`send_navigation_target_调用说明.md`
- `docs/plan.md`：开发计划（必须）

---

## 8. 维护者

- 项目负责人：`@<GitHub-ID>`（待填写）
