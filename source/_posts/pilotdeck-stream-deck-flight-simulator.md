---
title: "PilotDeck：让 Stream Deck 成为你的飞行仪表盘"
excerpt: "PilotsDeck 是一款免费开源的 Elgato Stream Deck 插件，可将你的 Stream Deck 变成功能强大的飞行模拟器控制面板，支持 MSFS、X-Plane 和 Prepar3D。"
date: 2026-05-29
categories:
- 其它
tags: [Stream Deck, 飞行模拟, PilotDeck, 工具]
---

## 引言

如果你是一名飞行模拟爱好者，你大概率拥有（或者正在考虑入手）一块 Elgato Stream Deck。这块小巧的可编程按键面板，最初是为直播者设计的，但它的可自定义 LCD 按键天然适合模拟飞行——想象一下，你的桌面上摆着一块面板，每个按键都能实时显示航向、高度、油量，按下就能操纵襟翼、起落架、灯光。这正是 **PilotsDeck** 要做的事情。

PilotsDeck 是由开发者 Fragtality 打造的一款完全免费、开源的 Stream Deck 插件。它能让你的 Stream Deck 同时充当飞行模拟器的输入设备和状态显示器，支持市面上三大主流飞行模拟平台：Microsoft Flight Simulator（MSFS 2020/2024）、X-Plane 12 和 Prepar3D v5。

## 什么是 PilotsDeck

PilotsDeck 的定位非常清晰：**它是连接 Stream Deck 硬件与飞行模拟器软件之间的桥梁。** 安装这个插件后，你可以在 Stream Deck 的每一个按键上配置两类功能：

1. **触发控制（Input）**：按下按键来执行模拟器中的操作，例如切换自动驾驶、调整航向、收放起落架、开关灯光等。
2. **状态显示（Display）**：在按键的 LCD 屏幕上实时显示模拟器中的数据，包括文字、图片、进度条（Bar）和弧形仪表（Arc）。

与市面上一些提供预设配置的商业方案不同，PilotsDeck 的设计哲学是**面向进阶用户**。它不会给你一个下拉菜单让你挑选"打开起落架"——你需要自己知道对应的变量名（SimVar）或命令（Command），然后手动配置到按键上。这看起来门槛不低，但带来的好处是极致的灵活性：**只要模拟器能暴露的变量和命令，PilotsDeck 都能读取和触发。**

从连接方式来说，PilotsDeck 根据不同模拟器使用不同的通信协议：

- **MSFS**：通过 SimConnect 直连，或可选通过 FSUIPC 连接。需要安装 MobiFlight 的 WASM 模块来访问更多变量。
- **X-Plane**：通过 UDP Socket 连接，支持远程连接（模拟器和 Stream Deck 可以不在同一台电脑上）。
- **Prepar3D / FSX**：必须通过 FSUIPC 连接（免费版即可）。

无论使用哪种模拟器，插件都会在模拟器启动后自动检测并连接，无需手动操作。更棒的是，你可以在不同模拟器之间无缝切换，无需重新配置。

## 核心功能

### 多样化的显示类型

PilotsDeck 的显示能力是它最大的亮点之一。每个按键可以配置为以下几种显示模式：

- **文字显示**：直接在按键上显示数值，比如当前航向 HDG 270°、高度 FL350、油量百分比等。支持自定义字体大小、颜色和格式化。
- **图片切换**：根据变量状态切换不同的图片。例如起落架收起时显示绿色图标，放下时显示红色图标。
- **进度条（Bar）**：以水平或垂直的进度条形式展示数值。可以自定义方向（左到右、下到上等）和颜色，支持阈值变色——比如油量低于 20% 时自动变红。
- **弧形仪表（Arc）**：模拟传统圆形仪表的显示方式。你可以设定起始角度、扫过角度、半径和厚度，打造出类似真实仪表盘的视觉效果。

Bar 和 Arc 都支持自定义偏移量来精确调整位置，还能叠加文字显示，实现"仪表 + 数值"的组合效果。

### 灵活的输入操作

在输入方面，PilotsDeck 支持多种命令类型：

- **SimConnect 命令**：直接调用 MSFS 的 SimConnect 事件。
- **Calculator/RPN 代码**：执行模拟器内部的 RPN（逆波兰）计算器代码，这是 MSFS 中最强大的控制方式，几乎可以操纵一切。
- **L-Var 操作**：直接读写本地变量，用于控制第三方插件飞机的特定功能。
- **X-Plane DataRef/Command**：原生支持 X-Plane 的数据引用和命令系统。
- **Lua 脚本**：插件内置了独立的 Lua 引擎，你可以编写脚本来实现更复杂的逻辑，比如条件判断、多步骤操作序列等。
- **vJoy 支持**：通过虚拟摇杆接口，将按键映射为摇杆按钮或轴。

对于 Stream Deck+ 用户，旋钮和触摸条（Encoder/Dial）也有完整支持，可以用来实现旋转调频、调高度等操作。

### 智能配置文件切换

这是 PilotsDeck 的杀手级功能之一。通过内置的 **Profile Manager**（配置管理器），你可以：

- 为不同的飞机创建专属配置。例如，给 A320 配置一套按键布局，给 Boeing 737 配置另一套。
- 设置**自动切换**：当你在模拟器中加载不同的飞机时，Stream Deck 上的配置会自动跟着切换。刚刚还在 A320 的 FCU 面板，换了架 737 后按键瞬间变成 MCP 布局。
- 配置默认配置文件：当没有匹配到特定飞机时，回退到默认布局。
- 模拟器关闭后自动切回指定的"待机"配置。

配置匹配基于模拟器返回的飞机路径字符串，支持模糊匹配，大小写不敏感。你可以在 Developer UI 的 Monitor 中查看当前飞机的匹配字符串，方便调试。

### Composite Action（组合动作）

较新版本引入了 Composite Action，允许你在单个按键上组合多种显示元素和操作。通过独立的 Action Designer UI 进行可视化配置，比传统的 Property Inspector 更加直观和强大。

## 安装配置

### 系统要求

- **操作系统**：仅支持 Windows（无 macOS/Linux 计划）
- **Stream Deck 软件**：v7.4 或更高版本
- **.NET 运行时**：.NET 10 x64 Desktop Runtime（安装程序会自动安装）
- **MSFS 用户**：需要安装 MobiFlight WASM Module 的最新版
- **P3D 用户**：需要 FSUIPC（免费版即可）
- **X-Plane 用户**：无额外依赖

### 安装步骤

1. 从 [GitHub Releases](https://github.com/Fragtality/PilotsDeck) 或 [Flightsim.to](https://flightsim.to/addon/35298/pilot-s-deck-a-streamdeck-plugin) 下载最新安装包。
2. 运行安装程序。它会自动检测并安装所需的 .NET 运行时和 Stream Deck 软件更新。
3. **重要提示**：不要以管理员身份运行安装程序、插件或 Stream Deck 软件。
4. 如果你使用 BitDefender 或其他杀毒软件，需要将安装程序和插件添加到白名单。这是社区中报告的头号问题来源。
5. 更新时直接运行新版安装程序即可，无需卸载旧版本。但注意：使用安装程序的"Remove"功能会同时删除所有自定义图片和脚本。
6. 安装完成后，在 Stream Deck 软件中就能看到 PilotsDeck 的 Action 列表，拖放到按键上即可开始配置。

### 基本配置流程

1. 打开 Stream Deck 软件，在右侧面板找到 PilotsDeck 的 Action。
2. 将 Action 拖到目标按键上。
3. 在 Property Inspector 中填写变量地址（如 `A:AUTOPILOT HEADING LOCK DIR, degrees`）和命令。
4. 配置显示样式：选择文字、图片还是仪表类型，设定颜色、字号等。
5. 启动模拟器，插件自动连接，按键开始工作。

## 使用场景

### 场景一：A320 飞行控制面板

这是 PilotsDeck 最经典的用法。使用一块 Stream Deck XL（32 键），你可以完整复刻 A320 的 FCU（飞行控制单元）面板：

- 第一行按键显示 SPD、HDG、ALT、VS 的当前值（文字+弧形仪表）
- 第二行按键对应推拉旋钮操作（managed/selected 模式切换）
- 底部按键控制 AP1/AP2、ATHR、LOC、APPR 等自动驾驶功能

社区已经提供了 FlyByWire A32NX 和 Fenix A320 的完整预制配置包，下载后通过 Profile Manager 一键导入即可使用。

### 场景二：系统监控面板

利用 Bar 和 Arc 显示功能，将一块 Stream Deck 变成系统监控面板：

- 双发动机的 N1/N2/EGT/FF 参数实时显示
- 液压系统压力、电气系统电压
- 油量百分比带阈值警告
- 襟翼位置指示器

### 场景三：多模拟器玩家的统一面板

如果你同时玩 MSFS 和 X-Plane，PilotsDeck 的跨模拟器支持非常实用：

- 配置一套通用布局，底层命令根据当前模拟器自动适配
- 模拟器之间切换时自动连接，无需手动操作
- 为每个模拟器设置不同的默认配置

### 场景四：配合 GSX Pro 的地面服务面板

社区甚至制作了 GSX Pro（地面服务插件）的专用配置，让你可以在 Stream Deck 上一键呼叫廊桥、加油车、餐车，大大提升地面操作的沉浸感。

## 与替代方案的对比

市面上还有一些类似的工具，简单对比：

| 特性 | PilotsDeck | FlightDeck | Flight Tracker |
|------|-----------|------------|----------------|
| 价格 | 免费开源 | 付费 | 免费开源 |
| 模拟器支持 | MSFS + X-Plane + P3D | MSFS + X-Plane | 仅 MSFS |
| 显示类型 | 文字/图片/Bar/Arc | 预制界面 | 文字/图片 |
| 自定义程度 | 极高（需手动配置） | 中等（预设为主） | 中等 |
| 配置文件切换 | 自动按飞机切换 | 支持 | 有限 |
| 上手难度 | 较高 | 较低 | 中等 |

PilotsDeck 的优势在于**极致的灵活性和零成本**，但代价是需要投入更多时间学习和配置。如果你喜欢折腾、追求完美定制，PilotsDeck 是不二之选；如果你更希望开箱即用，FlightDeck 可能更适合。

## 总结

PilotsDeck 是飞行模拟社区中一款不可多得的工具。它将一块普通的 Elgato Stream Deck 变成了功能强大的飞行仪表盘和控制面板，而且完全免费。虽然它的学习曲线不算平缓，需要你了解 SimVar、DataRef 等模拟器底层概念，但一旦配置完成，它带来的沉浸感和操控效率提升是巨大的。

对于那些已经拥有 Stream Deck 的模拟飞行玩家来说，PilotsDeck 几乎是必装插件。对于正在考虑购入 Stream Deck 的飞友，PilotsDeck 的存在本身就是一个有力的购买理由。

**项目地址：** [GitHub - Fragtality/PilotsDeck](https://github.com/Fragtality/PilotsDeck)
**下载地址：** [Flightsim.to](https://flightsim.to/addon/35298/pilot-s-deck-a-streamdeck-plugin) | [X-Plane.to](https://x-plane.to/file/179/pilot-s-deck-a-streamdeck-plugin)
**社区讨论：** [MSFS 论坛](https://forums.flightsimulator.com/t/introducing-pilots-deck-a-streamdeck-plugin/525266) | [AVSIM 论坛](https://www.avsim.com/forums/topic/620696-introducing-pilots-deck-a-streamdeck-plugin-for-msfs/) | [X-Plane.Org 论坛](https://forums.x-plane.org/files/file/83196-pilotsdeck-another-streamdeck-plugin/)
