# Van Step

> 将 SmartisanOS 的 **One Step** 侧边栏和 **BigBang** 文字拆分功能，以 LSPosed 模块的形式带入 HyperOS。

[![平台](https://img.shields.io/badge/platform-Android%2015--16-3DDC84?logo=android&logoColor=white)](#环境要求)
[![HyperOS](https://img.shields.io/badge/HyperOS2.0%20%7C%203.0-FF6900)](#环境要求)
[![框架](https://img.shields.io/badge/framework-LSPosed-6200EE)](https://github.com/LSPosed/LSPosed)
[![许可证](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

Smartisan OS 已经远去，但它最好的两个创意不应被遗忘。Van Step 在现代小米 HyperOS 之上还原了 One Step 侧边栏——将任何内容拖拽到任何地方——以及 BigBang——点按文字，即刻拆词。它通过 Hook SystemUI 和系统启动器来实现.

## 功能

### 侧边栏

从屏幕边缘滑入，即可查看最近的图片、文件和剪贴板历史。将条目拖拽到应用图标上即可分享，或拖拽到窗口槽位即可在其中打开。

- **最近图片 / 文件 / 剪贴板**，各自记住上次滚动位置
- **拖拽分享**至应用快捷方式，每次命中时提供触觉反馈
- **媒体控制**位于顶部栏，重新打开时定位到你上次使用的页面
- **任务切换**和自由窗口控制，包括主/副分屏互换

### BigBang

将任意文本炸裂为可点按的词语碎片，忠实还原原版体验：碎片先收缩至你的触摸点，然后向外迸发就位。

- 中英文混合分词，标点符号作为独立的窄碎片呈现
- 跨碎片拖拽进行范围选择；长按可拖出选区
- 复制、分享，或将选区直接拖入其他应用
- 以悬浮面板形式覆盖在变暗的宿主应用之上，而非全屏页面

### 手势触发方式

| 触发方式 | 工作原理 |
| --- | --- |
| 大面积按压 | 读取厂商触控 HAL 的接触密度信号——参见 [触控守护进程](#触控守护进程) |
| 双指长按 | 纯 `MotionEvent` 几何计算 |
| 长按回退 | 可选方案，用于面积信号不可用时 |
| 角落 / 边缘滑动 | 可从屏幕边缘自定义入口 |

模块设置中提供按应用黑名单和可调节的长按时长。

## 环境要求

- Android 15 – 16（`minSdk 35`，`targetSdk 36`）
- HyperOS  2.0 / 3.0（已在小米 15 Pro、 和 K40 上测试）
- 搭载 Zygisk 的 [LSPosed](https://github.com/LSPosed/LSPosed)
- Root 权限（Magisk / KernelSU / APatch）——持久化设置和触控守护进程均需要
！！！在使用前请备份数据 硬盘有价 数据无价！！！
## 安装

1. 安装 APK 并在 LSPosed 中启用 **Van Step**。
2. 启用以下作用域，然后**重启**：

   | 作用域 | 用途 |
   | --- | --- |
   | `系统框架`（`android`、`system`） | 窗口管理、重启策略、手势 Hook |
   | `系统界面`（`com.android.systemui`） | 侧边栏本体 |
   | `启动器`（`com.miui.home`） | 从桌面触发 |

   system_server 的 Hook 仅在开机时安装，因此重启是必须的——仅重启 SystemUI 不会安装这些 Hook。

3. 在 Root 管理器中授予本应用 **Root 权限**。设置通过 `su` 写入，如果没有 Root 权限，所有开关都会静默回退。

### 触控守护进程

大面积按压触发方式需要一个小型 Root 守护进程，因为触控驱动虽然声明了 `ABS_MT_TOUCH_MAJOR`，但从未实际发出该值——接触面积信息仅存在于厂商触控 HAL 内部。`scripts/onestep-touchd.sh` 通过 uprobe 读取该信息，并将判定结果以系统属性的形式发布。

将其安装为 Magisk/KernelSU 模块，以便每次开机自动启动：

```sh
adb push scripts/module/module.prop scripts/module/service.sh /data/local/tmp/
adb push scripts/onestep-touchd.sh scripts/onestep-touchd-stop.sh /data/local/tmp/
adb push scripts/deploy-module.sh /data/local/tmp/
adb shell su -M -c 'sh /data/local/tmp/deploy-module.sh'

### 遇到问题
