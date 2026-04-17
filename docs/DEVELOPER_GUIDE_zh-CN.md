# FlClash 开发入门文档

## 1. 文档目标

这份文档面向准备参与 FlClash 开发的同学，目标不是介绍产品功能，而是帮助你尽快完成下面几件事：

- 把项目拉起并建立基本运行环境
- 看懂项目的代码组织与运行链路
- 知道改一个功能时应该从哪里下手
- 知道常见修改后需要执行哪些生成、构建和验证动作
- 避免在 Flutter、Go Core、Rust Helper 的协作边界上踩坑

建议先阅读：

- [docs/PROJECT_ANALYSIS_zh-CN.md](D:\work\demo\FlClash\docs\PROJECT_ANALYSIS_zh-CN.md)

这份“开发入门文档”是在项目分析文档基础上继续向“可上手开发”方向扩展的。

## 2. 先建立整体认知

在开始写代码前，建议先接受一个事实：FlClash 不是单一语言工程，而是一个多层组合项目。

你可以把它理解成三层：

1. Flutter 应用层
   负责 UI、交互、配置编辑、状态同步、持久化、平台外壳能力。
2. Go Core 层
   负责代理内核封装、配置应用、日志、流量、连接、Provider 等核心能力。
3. 平台辅助层
   主要是桌面相关的插件和 Windows 下的 Rust Helper。

一个典型功能请求，通常会落在以下几种情况之一：

- 只改界面或交互：只需要改 `lib/views/`、`lib/widgets/`、`lib/providers/`
- 改业务状态流：一般需要同时看 `lib/controller.dart`、`lib/providers/`、`lib/models/`
- 改代理行为或 core 输出：需要进入 `lib/core/` 和 `core/`
- 改 Windows 启动或服务行为：可能涉及 `services/helper/`

## 3. 环境准备

## 3.1 必备工具

从仓库结构和构建脚本来看，至少需要准备：

- Flutter
- Dart
- Go
- Git

按平台不同，还需要额外工具：

- Windows 开发
  - Visual Studio / C++ 桌面开发环境
  - GCC
  - Inno Setup
- Android 开发
  - Android Studio 或 Android SDK
  - Android NDK
- Linux 开发
  - `libgtk-3-dev`
  - `libayatana-appindicator3-dev`
  - `libkeybinder-3.0-dev`
  - `ninja-build`
- macOS 开发
  - Xcode
  - 某些打包场景需要 `npm install -g appdmg`

如果你要完整参与多平台构建，还需要 Rust 环境，因为 Windows Helper 是用 Rust 构建的。

## 3.2 子模块初始化

这个项目依赖多个 Git 子模块，首次拉取后一定先执行：

```bash
git submodule update --init --recursive
```

如果这一步没做，后续你会遇到这些典型问题：

- `core/Clash.Meta` 不完整
- `plugins/tray_manager` 缺失
- `plugins/flutter_distributor` 缺失
- 构建脚本无法正常工作

## 3.3 Flutter 依赖安装

首次进入项目后，执行：

```bash
flutter pub get
```

如果后续你更新了依赖或切换分支，也建议重新执行一次。

## 4. 首次本地跑通建议

## 4.1 先区分“开发调试”和“完整打包”

你开发这个项目时，要先分清两件事：

- `flutter run`
  更适合 UI 开发、交互调试、状态调试。
- `dart .\setup.dart ...`
  更适合完整构建、生成 core 产物、打包发布。

如果只是调整 Flutter 页面，优先考虑 `flutter run`。
如果你改动了 core、helper 或最终发布链路，优先考虑 `setup.dart`。

## 4.2 推荐的首次跑通顺序

建议按下面顺序进行，而不是一上来直接全量打包：

1. 初始化子模块
2. 执行 `flutter pub get`
3. 先确认 Flutter 环境正常
4. 了解目标平台下 core 是否已经有可用产物
5. 如果没有，再用 `setup.dart` 生成对应平台产物
6. 再开始 `flutter run`

对 Windows 开发者，一个比较实用的顺序是：

```bash
git submodule update --init --recursive
flutter pub get
dart .\setup.dart windows --arch amd64
flutter run -d windows
```

对 Android 开发者，可以参考：

```bash
git submodule update --init --recursive
flutter pub get
dart .\setup.dart android
flutter run
```

说明：

- `setup.dart` 会负责构建 core，并处理不同平台的输出整理。
- 对于发布场景，它还会进一步打包应用。
- 如果你当前只做 Flutter 层改动，不一定每次都要重新跑完整 `setup.dart`。

## 5. 项目里最重要的几个入口

## 5.1 应用入口

- `lib/main.dart`

这里是 Flutter 启动入口。你可以从这里看清：

- 全局初始化从哪里开始
- `globalState` 何时初始化
- `Application` 何时挂载
- 初始化失败时的兜底逻辑

## 5.2 应用装配入口

- `lib/application.dart`

这里是整个应用树的拼装中心。开发时如果你遇到：

- 桌面/移动端差异行为
- 主题/本地化问题
- 网络变化触发的状态刷新
- 托盘、窗口、快捷键、VPN 容器问题

通常都应该先看这里。

## 5.3 应用调度中心

- `lib/controller.dart`

这是开发时最值得优先熟悉的文件之一。很多“按钮点击后到底发生了什么”的问题，都能顺着 `AppController` 找到答案。

尤其建议关注这些职责：

- 应用初始化
- 启停代理
- Profile 应用与配置更新
- 更新检查
- 托盘同步
- 备份恢复
- 页面切换

## 5.4 Core 调用入口

- `lib/core/controller.dart`
- `lib/core/service.dart`

这两处决定了 Flutter 如何和 Core 通信。

你可以简单记成：

- `CoreController` 是 Flutter 侧统一调用入口
- `CoreService` 是桌面端的进程与 Socket 通信实现

如果你改了 core 接口，一般 Dart 侧至少要看这两个文件。

## 5.5 Core 事件监听入口

- `lib/manager/core_manager.dart`

这里负责把 core 的回调结果转回 Riverpod 状态，非常关键。

常见现象与这里的关系：

- 延迟测试结果刷新不出来
- 日志页没数据
- 连接列表没更新
- core crash 后 UI 状态不对

## 6. 常见目录应该怎么读

## 6.1 改界面

优先看：

- `lib/views/`
- `lib/widgets/`
- `lib/pages/`

建议路径：

- 先在 `lib/views/` 找功能页面
- 再顺着页面里引用的组件进入 `lib/widgets/`
- 如果是页面导航切换问题，再回到 `lib/pages/home.dart`

## 6.2 改状态

优先看：

- `lib/providers/`
- `lib/models/`
- `lib/controller.dart`

通常判断方法是：

- 如果你只是“显示什么内容”，先找 Provider
- 如果你要“触发某个业务动作”，多半会经过 Controller
- 如果你要“新增一个配置项”，通常会同时改 Model + Provider + 持久化

## 6.3 改配置或 Profile 流程

优先看：

- `lib/models/config.dart`
- `lib/models/profile.dart`
- `lib/providers/config.dart`
- `lib/database/`
- `lib/controller.dart`

这类需求经常牵涉：

- 配置对象结构
- 持久化
- 页面展示
- 最终传入 core 的配置结果

## 6.4 改内核行为

优先看：

- `lib/core/`
- `core/action.go`
- `core/server.go`
- `core/` 下其他实现文件

常见场景：

- 新增一个 core 方法
- 修改 Dart 到 Go 的参数协议
- 调整返回结果
- 增加事件回调

这类改动要特别注意 Dart 和 Go 两边协议同步。

## 7. 推荐阅读顺序

如果你是第一次正式接手开发，我建议的顺序是：

1. `pubspec.yaml`
2. `lib/main.dart`
3. `lib/application.dart`
4. `lib/controller.dart`
5. `lib/pages/home.dart`
6. `lib/providers/`
7. `lib/models/`
8. `lib/core/controller.dart`
9. `lib/core/service.dart`
10. `lib/manager/core_manager.dart`
11. `core/action.go`
12. `core/server.go`
13. `setup.dart`

这样读的好处是：

- 先建立 Flutter 侧总体认知
- 再看状态流
- 最后看 Go Core 和构建系统

## 8. 你开始开发时最常见的 4 类任务

## 8.1 新增一个页面上的设置项

通常需要改：

- `lib/models/` 中的配置对象
- `lib/providers/` 中对应的状态 Provider
- `lib/views/` 中的设置页面
- `preferences` 存储链路

如果这个设置项最终要影响 core，还要继续改：

- 配置拼装逻辑
- `CoreController` 调用参数

## 8.2 新增一个 core 能力

通常需要改：

- `core/action.go`
- `core/` 中实际实现
- `lib/core/interface.dart` 或相关接口定义
- `lib/core/controller.dart`
- 视图层调用方

建议做法：

1. 先在 Go 侧定义动作和返回结果
2. 再在 Dart 侧补接口
3. 最后接 UI

不要反过来，不然很容易先把 UI 写好，结果协议没想清楚。

## 8.3 修改某个页面的展示逻辑

一般优先检查：

- 数据来自哪个 Provider
- Provider 是否来自数据库、配置还是 core
- 页面是否只是渲染问题，还是状态本身没更新

不要一上来就改 UI，有时候真正的问题在 `CoreManager` 或 `AppController`。

## 8.4 改构建或发布流程

优先看：

- `setup.dart`
- `plugins/flutter_distributor`
- `services/helper/`

这类问题往往不是 Flutter 页面层能解决的，直接从打包脚本入手更快。

## 9. 代码生成什么时候要跑

项目中使用了：

- `freezed`
- `json_serializable`
- `riverpod_generator`
- `drift_dev`

这意味着你改了以下内容后，通常需要重新生成代码：

- 带 `freezed` 的数据模型
- JSON 序列化对象
- Riverpod 注解 Provider
- Drift 数据表或 DAO

常用命令：

```bash
dart run build_runner build --delete-conflicting-outputs
```

开发中如果频繁调整模型，也可以考虑：

```bash
dart run build_runner watch --delete-conflicting-outputs
```

什么时候要想到它：

- `generated/` 目录没更新
- 编译报找不到某个 `.g.dart` / `.freezed.dart`
- Provider 代码和声明不一致
- Drift 表结构修改后报错

## 10. 常见调试思路

## 10.1 UI 不更新

先排查：

- 对应页面 `watch` 的是不是正确 Provider
- Provider 的值是否真的变化
- 是否只是 `read` 没有 `watch`
- 变更是否经过了 `notifier.update` 或 `value =`

## 10.2 点了按钮没反应

先排查：

- 页面点击事件有没有进入 `AppController`
- `AppController` 有没有继续触发 Provider 或 Core 调用
- Core 是否返回错误但只写到了日志里

## 10.3 Core 状态异常

优先看：

- `lib/manager/core_manager.dart`
- `lib/core/service.dart`
- `core/action.go`
- `core/server.go`

重点判断是：

- Flutter 没发请求
- Go 没收到请求
- Go 返回了但 Dart 没接到
- 事件到了但没写回状态

## 10.4 桌面端行为和 Android 不一致

这通常是正常现象，因为项目本来就是分平台适配。

优先检查：

- `application.dart` 中不同平台挂载了什么 Manager
- `CoreController` 选用了哪种实现
- 目标功能是否仅在桌面或仅在 Android 上存在

## 11. 修改前后的自检清单

每次改完功能，建议至少问自己这些问题：

1. 这次修改属于 Flutter UI、业务状态、Core 协议，还是构建链路？
2. 我有没有只改了一边，忘了同步另一边？
3. 这个改动需要重新跑 `build_runner` 吗？
4. 这个改动是否需要重新构建 core？
5. 桌面端和 Android 的行为是否都被考虑到了？

## 12. 我建议的开发节奏

如果你准备持续开发这个项目，我建议采用下面的节奏：

1. 先只跑通一个目标平台
   建议优先选你本机最容易调试的平台。
2. 先熟悉 Flutter 层
   不要一开始就深入 Go Core。
3. 遇到状态问题再进入 Provider 和 Controller
4. 遇到协议问题再进入 `lib/core/` 和 `core/`
5. 最后再看 `setup.dart` 和打包系统

原因很简单：

- 大部分日常开发改动其实发生在 Flutter 层
- Go Core 和构建系统更适合在你理解整体链路后再深入

## 13. 新功能开发的推荐落点

如果你要做新功能，可以先用下面这张对照表判断入口：

- 新页面或布局
  - `lib/views/`、`lib/widgets/`
- 新配置项
  - `lib/models/`、`lib/providers/`、`lib/controller.dart`
- 新数据库字段或实体
  - `lib/database/`、`lib/models/`
- 新 core 方法
  - `core/` + `lib/core/`
- 新托盘或桌面行为
  - `lib/manager/`、`plugins/`
- 新打包逻辑
  - `setup.dart`

## 14. 当前项目开发时最值得注意的坑

- 这是多语言工程，不要只盯着 Flutter 层。
- 子模块没有初始化时，很多错误看起来像“项目坏了”，其实只是依赖不完整。
- 配置、状态、UI、Core 之间经常是串联关系，不要只改其中一个点。
- 桌面端和 Android 的运行链路不同，调试时一定要明确当前平台。
- 修改 model、provider、database 后，别忘了代码生成。
- 修改 Go Core 后，别忘了对应平台可能需要重新构建产物。

## 15. 建议你第一次开发时做的练手任务

如果你想先熟悉项目而不是马上做复杂需求，我建议先做一个小练习：

1. 在设置页新增一个仅影响 UI 的布尔开关
2. 让它持久化
3. 让某个页面根据这个开关调整显示

这个练习会让你顺着走通：

- Model
- Provider
- 持久化
- 页面渲染
- 调试验证

这是理解项目最划算的一条路径。

## 16. 一句话行动建议

如果你现在就要开始开发，最实用的顺序是：

先把 `flutter pub get + git submodule update --init --recursive` 跑通，再选一个目标平台完成首次启动，然后从 `lib/main.dart -> lib/application.dart -> lib/controller.dart -> lib/views/` 这条线开始读代码。

