# Meituan Challenge Robot Code Documentation

美团挑战赛机器人代码说明。三个模块分别提供中文、英文 Markdown 和 PDF，说明主程序调度、电池视觉抓放及 Socket 通信的实际执行流程。

Bilingual documentation for the supplied robot program: main-program orchestration, vision-guided battery handling, and socket communication. Each module has Chinese and English Markdown and PDF versions.

| 模块 / Module | 中文 | English | 中文 PDF | English PDF |
| --- | --- | --- | --- | --- |
| 主程序 / Main program | [阅读](docs/zh/01-main-program.md) | [Read](docs/en/01-main-program.md) | [下载](docs/pdf/01-main-program-zh.pdf) | [Download](docs/pdf/01-main-program-en.pdf) |
| 电池任务 / Battery handling | [阅读](docs/zh/02-battery.md) | [Read](docs/en/02-battery.md) | [下载](docs/pdf/02-battery-zh.pdf) | [Download](docs/pdf/02-battery-en.pdf) |
| 通信模块 / Communication | [阅读](docs/zh/03-socket-communication.md) | [Read](docs/en/03-socket-communication.md) | [下载](docs/pdf/03-socket-communication-zh.pdf) | [Download](docs/pdf/03-socket-communication-en.pdf) |

## Materials and scope

- [Modul.docx](Modul.docx): `Modul1`, shared definitions and the main loop.
- [battery.docx](battery.docx): `Battery`, coordinate acquisition and single/multiple battery handling.
- [connect_socket.docx](connect_socket.docx): `Connect_socket`, camera communication, data parsing and PLC helpers.
- [task1.jpg](task1.jpg) and [task2.jpg](task2.jpg): VisionMaster workflow screenshots.

Source revision: [68e262b](https://github.com/lkkkkk667/meituan/commit/68e262bbedcf2269ba3a7321a045f97e96f185b6). Documentation date: 2026-10-13.

说明按可执行语句解释行为，保留注释与执行语句不一致、初始化依赖缺失等需核对事项。当前代码采用 ABB RAPID、视觉结果解析和预设抓放流程；没有材料证明其实现 VLA、Diffusion Policy 或完整自主换电系统。原始材料保持原样。

The documentation follows executable statements and records relevant comment/code discrepancies and missing initialization dependencies. The supplied implementation uses ABB RAPID, visual-result parsing and predefined handling sequences; the materials do not establish a VLA/Diffusion Policy implementation or a complete autonomous battery-swapping system. Original materials are unchanged.

## Verification status

已完成文档与源代码的静态对照。未执行控制器编译、RobotStudio 仿真、网络联调或真机测试；这些 DOCX 不构成完整可直接运行的工作站。

Documentation has been statically cross-checked against the supplied source. Controller compilation, RobotStudio simulation, network integration and physical-robot testing have not been performed. The DOCX files do not constitute a complete ready-to-run workstation.
