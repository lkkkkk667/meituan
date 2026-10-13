# 主程序与系统调度说明

美团挑战赛机器人代码说明 01 | 中文版 | 2026 年 10 月 13 日

## 1 文档概述

本文说明 `Modul.docx` 中的 ABB RAPID 模块 `Modul1`：它集中定义跨模块共享的数据、机器人示教点和运行状态，并通过 `main` 持续调用电池任务。配套的 `Battery` 模块负责视觉引导抓放，`Connect_socket` 模块负责相机和 PLC 通信。三份说明分别介绍调度、作业和通信，按同一代码版本对应阅读。

当前程序体现的是视觉定位配合预设运动与顺序控制。材料中没有 VLA、Diffusion Policy、模型训练、自主导航或完整换电机构控制实现，本文不将这些能力列为已实现功能。

## 2 模块分工与调用关系

| 源文件与模块 | 主要职责 | 关键入口 |
| --- | --- | --- |
| Modul.docx / Modul1 | 共享变量、示教点、主循环与任务调度 | main、Task_Set_2 |
| battery.docx / Battery | 取视觉坐标、选择任务、执行抓放 | Battery_work |
| connect_socket.docx / Connect_socket | 建立连接、收发字符串、解析坐标及 PLC 报文 | Connect2、ProcessReceivedData |

主调用链如下。`initial` 是外部依赖；三个提供的模块中均没有找到它的定义。

```text
main
  -> initial [定义未包含在三个文档中]
  -> StartProgram := TRUE
  -> WHILE StartProgram
       -> Task_Set_2
            -> Battery_work
                 -> Battery_get_values
                      -> Connect2
                           -> SocketConnectCamara
                                -> ProcessReceivedData
                 -> Battery_work_2 或 Battery_work_1
```

`Task_Set_2` 是调度过程的名称，并不表示主程序只执行视觉 Task 2。实际选择单电池或多电池作业的逻辑位于 `Battery_work`。

## 3 主程序执行流程

1. `main` 首先调用 `initial`，随后等待 0.1 秒。由于初始化过程没有包含在材料中，其具体动作和初始状态不能从当前文件确定。
2. 程序将 `StartProgram` 置为 `TRUE`，进入 `WHILE StartProgram DO` 循环。
3. 每轮执行 `Task_Set_2`。该过程打印开始提示，调用 `Battery_work`，返回后打印结束提示。
4. 当前三个模块内没有看到在正常任务结束时将 `StartProgram` 改为 `FALSE` 的语句，因此主循环会继续进入下一轮。

“Finish Task” 表示被调用过程已经返回，不构成电池成功抓取、正确放置或赛事任务完成的检测结果。无目标时的提前返回同样可能走到该提示。

## 4 数据与设备依赖

| 数据类别 | 代表标识符 | 在执行链中的用途 |
| --- | --- | --- |
| 通信数据 | socket1、socket2、input_string、send_string、receive_string_copy | 区分相机/PLC 通道并传递请求和回复 |
| 视觉结果 | nval1 至 nval6 | 保存单目标或三组目标的解析数值 |
| 任务与通信状态 | StartProgram、socket_start、startPLC、CamaraFinish | 控制主循环、通信分支及解析结束标记 |
| 视觉补偿 | diamondX、diamondY | 获取坐标后加入公共位置补偿 |
| 机器人目标点 | Redy_Camara_Diamonds、D_pick、D_put_Red/Blue/Green/Yellow | 提供拍照参考、抓取高度参考和放置点 |
| 工具与工件坐标 | tool_D、wobj0、wobjDiamond | 供运动指令指定工具和参考坐标系 |

`Modul1` 还保留多组 W、P、B 前缀目标点及其他状态变量；这些定义没有在当前可见的电池主调用链中全部使用，不能据此推断相应作业也会运行。现场标定坐标属于具体工作站配置，不应直接复制为另一台机器人的通用参数。

## 5 集成与验证条件

这三个 DOCX 是代码文本载体，不是完整可直接部署的 RobotStudio 工作站或控制器备份。在目标工作站中核对以下条件后，才能判断程序是否可运行：

1. 补齐并核对 `initial` 的实际定义和初始化顺序，以及 RAPID 模块之间共享数据的可见性。
2. 核对 `tool_D`、`wobjDiamond` 和数字输出 `IIO_do2` 的工作站配置；这些配置没有在三份材料中完整给出。确认 `wobj0` 对应的实际参考框架。
3. 核对相机坐标、`diamondX/diamondY`、工具方向、抓取高度与示教放置点之间的一致性。
4. 在对应控制器/仿真环境中检查语法与依赖，然后由现场人员进行受控单步和低速验证。
5. 分别验证无目标、单红色目标、三色目标、通信异常和重复循环情况下的执行行为。

本说明完成的是代码静态解读。尚无随附编译记录、仿真结果、真机日志或成功率统计，因此不提供“已编译通过”“已完成真机验证”或性能指标结论。

## 6 源码定位

源版本：`lkkkkk667/meituan`，提交 `68e262bbedcf2269ba3a7321a045f97e96f185b6`。

- `Modul.docx`：`MODULE Modul1` 下的变量与目标点定义；`PROC main()`；`PROC Task_Set_2()`。
- `battery.docx`：`PROC Battery_work()` 与 `PROC Battery_get_values()`。
- `connect_socket.docx`：`PROC Connect2()`、`PROC SocketConnectCamara()`、`PROC ProcessReceivedData()`。

配套说明：[电池任务](02-battery.md) · [通信模块](03-socket-communication.md)。
