# Socket 通信与视觉数据解析说明

美团挑战赛机器人代码说明 03 | 中文版 | 2026 年 10 月 13 日

## 1 文档概述

`connect_socket.docx` 中的 `Connect_socket` 模块负责机器人与视觉系统、PLC 之间的通信。相机分支发送字符串命令、接收视觉结果并解析成共享数值；PLC 分支提供寄存器读写与握手控制。电池抓放主链显式设置相机分支，不能因为文件中存在 PLC 过程就认为每轮电池任务都会执行 PLC 操作。

`socket1/socket2`、收发字符串、数值字段和多数状态变量在 `Modul1` 中定义；通信文件开头的同名声明主要处于注释状态。

## 2 连接分派与相机请求

`Connect2` 根据两个布尔量选择互斥分支：

| socket_start | startPLC | 行为 |
| --- | --- | --- |
| TRUE | FALSE | 建立 socket1 连接并调用 SocketConnectCamara |
| FALSE | TRUE | 建立 socket2 连接并调用 ConnectPLC |
| 其他组合 | 其他组合 | 当前 ELSE 分支不执行操作 |

原程序包含工作站内设备地址和端口。部署时应根据实际网络配置核对；本说明不复制内网地址，也不将它们作为通用配置。

相机正常处理链如下：

```text
Battery_get_values
  -> send_string := input_string
  -> socket_start := TRUE; startPLC := FALSE
  -> Connect2
       -> 建立相机连接
       -> SocketConnectCamara
            -> SocketSend 发送 send_string
            -> SocketReceive 接收到 receive_string_copy
            -> 保存 receive_string 和字符串长度
            -> 关闭两个 socket
            -> ProcessReceivedData
                 -> 重置 nval1..nval6
                 -> 解析字段并打印调试值
                 -> CamaraFinish := TRUE
```

`Getvalues` 用于多目标坐标请求，`Redvalues` 用于单红色目标请求。当前代码按请求创建连接并在接收后关闭连接；没有显示可直接称为持续长连接的相机会话管理。

## 3 数据字段与解析过程

| 字段 | 三色流程的含义 | 单红色流程中的用途 |
| --- | --- | --- |
| nval1、nval2 | 蓝色 X、Y | 红色 X、Y，供抓放使用 |
| nval3、nval4 | 绿色 X、Y | 第三个字段的注释标为角度；抓放未使用它调整姿态 |
| nval5、nval6 | 黄色 X、Y | 无对应单色抓放用途 |

`ProcessReceivedData` 先将六个数值置零，再从 `receive_string_copy` 提取字段。代码尝试跳过第一个逗号之前的冒号前缀，用逗号与分号区分字段，并在组间处理双分号；每个数值通过 `StrToVal` 写入共享变量。

从分支结构可推导出的格式意图如下，它们是接口核对示意，不是已通过控制器验证的协议样本：

```text
单目标：X,Y,R
三目标：Xb,Yb;;Xg,Yg;;Xy,Yy
```

实际输出是否带前缀、末尾分隔符、空字段或角度，必须与 VisionMaster 的格式化节点和真实回复逐项对齐。当前没有完整视觉工程配置或抓包数据，不应声称解析器已支持所有形式或异常数据。

`ok` 接收了 `StrToVal` 的返回值，但代码没有根据它逐项拒绝无效结果。`CamaraFinish = TRUE` 表示过程到达末尾，不等于已确认所有字段转换成功。缺少分隔符时的 `StrFind` 返回行为和后续切片边界也需在目标 RobotWare 版本上验证。

## 4 PLC 寄存器分支

`ConnectPLC` 将读写数量设为 1，并根据共享变量 `address` 选择操作：

| 条件或寄存器 | 当前代码行为 |
| --- | --- |
| address = 3000 | 将 WriteStart 设为 3000，并写入值 1 |
| address 在 2101 至 2104 | 向 2100 写入 address - 2100 |
| 寄存器 2020 | 写入值 1，作为代码中注释描述的同步标志 |
| 寄存器 2000 | 持续读取，直到 Readnum{1} 不为零 |

上述设备含义来自代码注释：3000 对应传送带气缸，2100 对应转台角度代号，2000 对应允许机器人动作的状态。真实 PLC 程序与寄存器映射没有随附，现场配置必须进一步核实。循环退出后仅设置了 `WriteStart = 2000`；相邻复位与写入语句处于注释状态，不能描述成“已经清零该寄存器”。

正常结束时，过程设置 `PlcFinish = TRUE`，复位 `address` 与 `startPLC`，并关闭两个 socket。若 2000 的读取值持续为零，当前等待循环没有显式的次数或时间上限。

## 5 报文辅助过程

- `ReadHoldingRegister` 组装 12 字节读取请求，功能码为 `0x03`，发送后接收字节数组，再调用 `ByteForWord`。
- `ByteForWord` 合并响应中的两个字节，当前循环只解出一个寄存器值，并赋给 `Readnum`。
- `WriteHoldingRegister` 调用 `WordForbyte` 打包写请求，然后发送并接收回复。
- `WordForbyte` 使用 `0x10` 写多个寄存器功能码，并按高低字节拆分地址、数量和数据；当前发送数组固定为 15 字节，与现有单寄存器使用方式相对应，不能直接扩展为任意数量寄存器写入。

该字节布局体现 Modbus TCP 风格的寄存器通信，但材料没有 PLC 接口规范、响应校验和联调记录。读取过程使用全局 `ReadStart`，报文中的单元标识固定；不应把过程签名中的参数都描述成已经完整驱动请求内容。

## 6 异常处理与集成核对

`Connect2` 的 `ERROR` 段复位通信选择标志与 `address`，并关闭 socket。当前未见显式重连次数、结构化错误回传或统一清除全部视觉结果的逻辑。`SocketConnectStatus` 和 `all_socket` 也存在于模块中，但没有出现在已提供的电池主链调用路径里。

集成时重点核对：相机请求与回复格式、分隔符搜索边界、数值转换失败后的处理、网络异常后旧值的隔离、PLC 响应字段及等待退出条件。运行前需补齐主程序说明列出的初始化、工具、工件坐标和 I/O 配置。

本说明是静态代码解释，未运行网络接口、未连接机器人或 PLC，也未执行控制器编译、仿真或真机测试。

## 7 源码定位

源版本：`lkkkkk667/meituan`，提交 `68e262bbedcf2269ba3a7321a045f97e96f185b6`。

`connect_socket.docx` 中的 `Connect2`、`SocketConnectCamara`、`ProcessReceivedData`、`ConnectPLC`、`ReadHoldingRegister`、`ByteForWord`、`WriteHoldingRegister`、`WordForbyte`；跨模块变量见 `Modul.docx`，相机命令设置见 `battery.docx`。

配套说明：[主程序](01-main-program.md) · [电池任务](02-battery.md)。
