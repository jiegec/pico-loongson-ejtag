
# Pico Loongson EJTAG

[ENGLISH](README.en.md)

欢迎加入！本项目专为龙芯嵌入式开发者爱好者打造，旨在重新实现龙芯官方EJTAG仿真器的核心功能，力求实现硬件功能完全对标且上位机软件兼容互通。

作为龙芯生态的共建者，我们深知龙芯3A6000等桌面级处理器性能比较亮眼，但其整机体积大、价格门槛高、与海外顶尖产品尚存差距的现状，让不少开发者望而却步（包括笔者本人）。而2K0300开发板的问世彻底改变了这一局面——这颗SoC不仅拥有媲美（超越）iMX6ULL等主流芯片的CoreMark性能，支持跑Linux/裸机，更以价格（百元级开发板）和小巧体积，为开发者打开了低成本进入LoongArch世界的通道。

然而，对于专注于裸机开发的嵌入式开发者而言，现有生态仍存在痛点：虽然2K0300可运行裸机程序，但官方调试工具高昂的售价成为了拦路虎。长期以来，龙芯的主要盈利点集中在桌面领域，对嵌入式生态投入有限（2K0300至今仅有第三方提供的残缺SDK，其IDE更是近乎讽刺地仅支持Windows平台）。我们希望通过这个项目证明：嵌入式开发者完全有能力为LoongArch的嵌入式开放生态开辟道路。

当前，固件基于逆向工程成果，主要参考对象为搭载20210129版本FPGA固件的官方调试工具和`la_dbg_tool_usb.exe`工具，并基于此独立实现了设备与上位机的通信协议、JTAG驱动。

***当前版本暂未实现高速内存读写功能（固件烧录因此不可用），后续将持续完善，敬请期待！***

## 实现方案

本固件基于树莓派RP2040-Zero开发板（成本极低，轻松复刻），具体引脚定义如下：

| RP2040 GPIO编号 | JTAG信号  | 备注 |
|----------------|----------|-----|
| 8              | #TRST    |     |
| 9              | TDI      |     |
| 10             | TDO      |     |
| 11             | TMS      |     |
| 12             | TCK      |     |
| 13             | #BRST    |     |
| 14             | #DINT    |     |
| 26             | VIO      | 模拟引脚 |

板载WS2812（GPIO16控制）作为状态指示灯，GPIO0/1上的UART0接口可输出开发调试信息。

- `src/lsejtag`与`inc/lsejtag`目录封装了与平台无关的核心逻辑（内部称其为`LS-EJTAG`协议栈，但并没有独立发布），负责处理上位机指令解析
- `src/ejtag_impl.c`包含硬件平台相关代码，移植至其他主控芯片时需重新实现

我们正在酝酿基于HPM5301主控的升级方案（硬件设计采用OSL-W-2.0协议开源，固件遵循GPL-3.0协议），重点是利用它的高速USB以图加快速度，敬请期待！

# 构建指南

我个人因为并不想安装体积比较大的SDK安装包，基于Windows环境手搓了构建流程，请按以下步骤配置开发环境：

## picotool安装
请将`picotool`完整安装至`C:\Program Files\CMake\picotool\`目录（内含CMake目标文件等必要组件）。`pioasm`工具也需同步安装。

**替代方案**：将安装目录路径添加至`CMAKE_PREFIX_PATH`环境变量，具体方式可依个人偏好选择。

## 环境配置
- `PICO_SDK_BASE`：设置为pico-sdk仓库的本地克隆路径（直接指向仓库根目录）
- `PICO_SDK_COMPILER_PATH`：指向MCU编译器所在目录（需包含`arm-none-eabi-gcc.exe`等可执行文件）

## VSCode编译流程
1. 通过Workspace Settings配置上述环境变量
2. 启动构建时选择"ARM GCC (Pico)"工具链
3. 正常情况下即可完成编译，若遇异常...祝君调试愉快

## clangd适配
因clangd/LLVM的设计缺陷（详见clangd/clangd#2289），需在clangd参数中添加：`--query-driver=.../your/gnu/toolchain/bin/arm-none-eabi-*.exe`以强制clangd识别GNU工具链路径（否则会误判为LLVM工具链导致头文件搜索失败）

# 许可

本项目除引用了其他项目有明确版权标识的文件之外，其他的代码均以 GPL-3.0-only 许可证发布。
