# FlClash 模块地图

## 1. 这份文档怎么用

这份“模块地图”不是完整 API 文档，而是面向开发时的定位手册。

你可以把它当成一张快速索引表：

- 想改页面，先看哪里
- 想改状态，先看哪里
- 想改 Core 通信，先看哪里
- 想判断某个问题属于 UI、Provider 还是 Core，先看哪里

本文重点覆盖三块：

- `lib/views/`
- `lib/providers/`
- `lib/core/`

## 2. 总体关系图

可以先把这三块理解成下面这条链：

`views` -> `providers/controller` -> `core`

更具体一点：

1. `lib/views/`
   负责页面与功能视图，是用户直接看到的部分。
2. `lib/providers/`
   负责应用状态、配置状态、数据库状态、派生状态，是 Flutter 侧的数据中枢。
3. `lib/core/`
   负责 Flutter 与 Go Core 的统一接口、协议封装、事件回流。

如果你发现某个功能“显示不对”，通常从 `views` 开始。
如果你发现某个功能“状态不对”，通常从 `providers` 开始。
如果你发现某个功能“调用 core 没生效”，通常从 `core` 开始。

## 3. `lib/views/` 模块地图

## 3.1 入口文件

- `lib/views/views.dart`

作用：

- 这是视图层的聚合导出文件。
- 可以快速看到当前有哪些一级功能视图被暴露给页面系统。

适合什么时候看：

- 你刚接手项目，想快速了解有多少主功能页。
- 你新增一个一级视图后，想确认导出入口。

## 3.2 一级页面文件

### `lib/views/about.dart`

作用：

- 关于页相关视图。

什么时候看：

- 改版本信息、项目介绍、外部链接、版权展示等。

### `lib/views/access.dart`

作用：

- 从命名看，偏权限、访问控制或相关入口页。

说明：

- 这是根据文件名和项目语义做的推断，具体行为建议结合页面引用处一起看。

### `lib/views/application_setting.dart`

作用：

- 应用设置主页面之一。

什么时候看：

- 改应用级偏好项。
- 新增一个设置入口。
- 调整设置页分组展示。

通常会联动：

- `lib/providers/config.dart`
- `lib/models/config.dart`
- `lib/controller.dart`

### `lib/views/backup_and_restore.dart`

作用：

- 备份恢复相关界面。

什么时候看：

- 改导出、恢复、覆盖策略等。

通常会联动：

- `lib/controller.dart`
- `lib/database/`
- 配置持久化逻辑

### `lib/views/developer.dart`

作用：

- 开发者工具或调试相关页面。

什么时候看：

- 想暴露调试开关、日志辅助能力、内部实验项。

### `lib/views/hotkey.dart`

作用：

- 快捷键配置相关界面。

通常会联动：

- `lib/providers/config.dart`
- `lib/manager/hotkey_manager.dart`

### `lib/views/logs.dart`

作用：

- 日志查看页面。

什么时候看：

- 日志显示异常
- 日志筛选/展示样式调整
- 排查 core 日志是否写回 UI

通常会联动：

- `lib/manager/core_manager.dart`
- `lib/providers/state.dart`

### `lib/views/resources.dart`

作用：

- 资源相关页面，通常可能和 Geo 资源、Provider 资源或下载资源有关。

### `lib/views/theme.dart`

作用：

- 主题配置、主题效果展示或颜色方案页。

通常会联动：

- `lib/providers/config.dart`
- `lib/state.dart`

### `lib/views/tools.dart`

作用：

- 工具集合页。

什么时候看：

- 新增非主线但实用的工具能力。

## 3.3 重点功能子目录

### `lib/views/dashboard/`

核心文件：

- `dashboard/dashboard.dart`

作用：

- 仪表盘主视图。
- 支持小组件布局、编辑模式、组件增删、仪表盘持久化。

为什么重要：

- 这是首页主视图之一，通常也是用户停留时间最长的页面。
- 从已有代码看，它还承担了“可编辑面板布局”的角色。

适合什么时候看：

- 改首页展示
- 新增 Dashboard 小组件
- 调整编辑/保存行为

联动重点：

- `appSettingProvider`
- 仪表盘相关 model
- dashboard widgets 子目录

### `lib/views/dashboard/widgets/`

作用：

- Dashboard 的具体卡片组件集合。

什么时候看：

- 你要新增一个仪表盘组件
- 你只想改某张卡片，不想碰整个 dashboard 页面

### `lib/views/profiles/`

核心文件：

- `profiles/profiles.dart`
- `profiles/add.dart`
- `profiles/edit.dart`
- `profiles/overwrite.dart`

职责拆解：

- `profiles.dart`
  - Profile 列表主页面
  - 选择、排序、菜单操作、同步、删除等
- `add.dart`
  - 新建 Profile
- `edit.dart`
  - 编辑已有 Profile
- `overwrite.dart`
  - 覆写或扩展某个 Profile 相关配置

为什么重要：

- Profile 是项目里最核心的业务实体之一。
- 订阅导入、配置切换、覆盖规则、导出文件，很多主线功能都会经过这里。

适合什么时候看：

- 改订阅管理
- 改配置文件导入/导出
- 改 Profile 菜单操作
- 改排序、同步、启用行为

联动重点：

- `lib/providers/database.dart`
- `lib/providers/config.dart`
- `lib/controller.dart`
- `lib/database/`

### `lib/views/proxies/`

核心文件：

- `proxies/proxies.dart`
- `proxies/providers.dart`
- `proxies/list.dart`
- `proxies/card.dart`
- `proxies/tab.dart`
- `proxies/setting.dart`
- `proxies/common.dart`

职责拆解：

- `proxies.dart`
  - 代理页主入口
- `providers.dart`
  - 代理页内部状态或局部 Provider
- `list.dart`
  - 列表视图
- `card.dart`
  - 卡片视图
- `tab.dart`
  - 分组/标签页切换
- `setting.dart`
  - 代理页专属展示设置
- `common.dart`
  - 代理页通用逻辑与组件

为什么重要：

- 这是与 core 交互最频繁的页面之一。
- 代理组、节点切换、延迟展示通常都集中在这里。

适合什么时候看：

- 改代理组 UI
- 改节点切换交互
- 改延迟显示
- 改代理页布局样式

联动重点：

- `lib/providers/state.dart`
- `lib/core/controller.dart`
- `lib/manager/core_manager.dart`

### `lib/views/config/`

核心文件：

- `config/config.dart`
- `config/general.dart`
- `config/network.dart`
- `config/dns.dart`
- `config/rules.dart`
- `config/scripts.dart`
- `config/advanced.dart`

职责拆解：

- `config.dart`
  - 配置页主入口
- `general.dart`
  - 常规配置
- `network.dart`
  - 网络相关设置
- `dns.dart`
  - DNS 相关配置
- `rules.dart`
  - 规则相关配置
- `scripts.dart`
  - 脚本相关配置
- `advanced.dart`
  - 高级配置

为什么重要：

- 几乎所有“调整 Clash 配置行为”的需求最终都会经过这里。

适合什么时候看：

- 增加一个配置项
- 调整配置页分组
- 修改某类配置的录入方式

联动重点：

- `lib/providers/config.dart`
- `lib/models/config.dart`
- `lib/core/controller.dart`

### `lib/views/connection/`

核心文件：

- `connection/connections.dart`
- `connection/requests.dart`
- `connection/item.dart`

职责拆解：

- `connections.dart`
  - 当前连接列表或连接管理视图
- `requests.dart`
  - 请求跟踪视图
- `item.dart`
  - 单条连接/请求的展示组件

为什么重要：

- 连接与请求数据通常来自 core 事件流，是排查代理行为的重要观察窗口。

适合什么时候看：

- 想调试请求跟踪
- 想改连接列表展示
- 想扩展请求详情信息

联动重点：

- `lib/manager/core_manager.dart`
- `lib/providers/state.dart`
- `lib/core/controller.dart`

## 3.4 `views` 层开发建议

如果你要改 `views`，建议先判断问题是哪一类：

- 只是布局或样式问题
  - 先看当前 `view` 文件和 `lib/widgets/`
- 页面显示的数据不对
  - 先看它读的是哪个 Provider
- 操作按钮点击后没反应
  - 顺着点击事件追到 `AppController` 或 `CoreController`

## 4. `lib/providers/` 模块地图

`providers` 是 Flutter 侧的状态中心，可以理解成“页面与底层逻辑之间的数据中间层”。

## 4.1 入口文件

- `lib/providers/providers.dart`

作用：

- 聚合导出 Provider 层。

什么时候看：

- 快速了解 Provider 模块整体暴露了哪些子文件。

## 4.2 配置状态核心

### `lib/providers/config.dart`

这是配置状态最关键的文件之一。

它主要负责：

- 应用设置 `AppSetting`
- 窗口设置 `WindowSetting`
- VPN 设置 `VpnSetting`
- 网络设置 `NetworkSetting`
- 主题设置 `ThemeSetting`
- 当前 Profile `CurrentProfileId`
- WebDAV 设置 `DavSetting`
- DNS 覆写 `OverrideDns`
- 快捷键动作 `HotKeyActions`
- 代理页样式 `ProxiesStyleSetting`
- Clash 配置补丁 `PatchClashConfig`
- 聚合配置 `configProvider`

为什么它重要：

- 几乎所有“设置类需求”的最终状态入口都在这里。
- 它还会把多个细粒度 Provider 重新聚合成一个完整 `Config` 对象。

适合什么时候看：

- 新增配置项
- 配置项不生效
- 页面保存设置后没持久化
- 某个设置在 UI 显示正常但应用后无效

联动重点：

- `lib/models/config.dart`
- `lib/controller.dart`
- `preferences` 存储逻辑

### 推荐阅读顺序

如果你要改设置类功能，建议顺序是：

1. `lib/models/config.dart`
2. `lib/providers/config.dart`
3. 对应 `lib/views/config/` 或设置页
4. `lib/controller.dart`

## 4.3 应用运行时状态核心

### `lib/providers/state.dart`

这是运行时状态的核心文件之一，也是最常用的 Provider 文件。

从已有内容看，它主要承载：

- 视图尺寸与模式
- 初始化状态
- 当前页面标签
- 排序编号
- IP 检测触发计数
- 返回拦截状态
- 代理分组
- 延迟数据源
- 系统 UI 样式
- Core 状态
- 通用查询状态
- Loading 状态
- 选中项状态
- 更新中状态
- 网络检测结果

为什么它重要：

- 很多页面状态、全局 UI 状态、Core 运行态都在这里汇总。
- 如果你遇到“页面逻辑没更新”或“状态同步不稳定”，这里通常要看。

适合什么时候看：

- 页面导航状态异常
- 代理分组显示不对
- Loading 不消失
- Core 连接状态不对
- IP 检测逻辑异常

联动重点：

- `lib/application.dart`
- `lib/pages/home.dart`
- `lib/manager/core_manager.dart`

## 4.4 数据库状态核心

### `lib/providers/database.dart`

这是数据库实体在 Flutter 层的主要投影层。

核心 Provider 包括：

- `profilesStream`
- `addedRuleStream`
- `Profiles`
- `Scripts`
- `GlobalRules`
- `ProfileAddedRules`
- `ProfileDisabledRuleIds`

为什么它重要：

- 它把数据库表和页面状态连接起来。
- 你看到的 Profile 列表、脚本列表、规则列表，很多不是直接查数据库，而是通过这里拿到。

适合什么时候看：

- Profile 列表不刷新
- 删除 Profile 后 UI 没更新
- 规则排序异常
- Script 新增后列表没变化

联动重点：

- `lib/database/`
- `lib/views/profiles/`
- `lib/views/config/rules.dart`
- `lib/controller.dart`

## 4.5 `generated/`

- `lib/providers/generated/*.g.dart`

作用：

- Riverpod 注解生成文件。

开发建议：

- 不要手改。
- 如果 Provider 声明改了却报生成错误，记得跑：

```bash
dart run build_runner build --delete-conflicting-outputs
```

## 4.6 `providers` 层开发建议

如果你在 Provider 层新增能力，通常要先判断它属于哪类：

- 配置型状态
  - 优先加到 `config.dart`
- 运行时状态
  - 优先加到 `state.dart`
- 数据库存储映射
  - 优先加到 `database.dart`

这个划分很重要，能避免把状态放乱。

## 5. `lib/core/` 模块地图

`lib/core/` 是 Flutter 与 Go Core 之间的协议层和适配层。

如果说 `views` 是表层，`providers` 是中层，那 `core` 就是 Flutter 侧最靠近底层能力的一层。

## 5.1 聚合入口

### `lib/core/core.dart`

作用：

- 统一导出 core 模块的关键文件。

什么时候看：

- 想快速知道 core 层有哪些组成。

## 5.2 统一调用入口

### `lib/core/controller.dart`

这是 Flutter 侧 Core 调用的总入口，重要程度非常高。

它主要负责：

- 根据平台选择实现
  - Android 使用 `coreLib`
  - 桌面端使用 `coreService`
- 初始化 Geo 数据
- 初始化 core
- 关闭 core
- 校验配置
- 更新配置
- 应用配置
- 获取代理组
- 切换代理
- 获取连接、流量、内存
- 获取 Provider、Geo 数据
- 启停日志监听

为什么它重要：

- 几乎所有 Flutter 到 Core 的业务动作最终都会经过这里。
- 当你怀疑“UI 调到了 Core 吗”，通常就是先看这里。

适合什么时候看：

- 配置应用不生效
- 代理切换无效
- 连接、流量、日志拿不到
- 想新增一个 Core 能力

## 5.3 协议抽象层

### `lib/core/interface.dart`

这是 Core 协议封装的抽象层。

它的职责包括：

- 定义 Core 应具备哪些能力
- 把 `ActionMethod` 封装成统一调用
- 对调用前等待、超时、日志监控做统一处理
- 提供 `init / updateConfig / setupConfig / getProxies` 等标准实现

为什么它重要：

- 这里定义了“Flutter 认为 Core 应该怎么被调用”。
- 你要新增一个 Core 方法时，这里往往是协议同步的关键文件。

适合什么时候看：

- 新增动作方法
- 调整请求参数格式
- 调整返回结果处理
- 排查某个 Core 调用为什么返回空值或超时

## 5.4 桌面端实现

### `lib/core/service.dart`

这是桌面端 Core 实现的关键文件。

它主要负责：

- 启动本地 Socket 服务端
- 启动 core 子进程
- 管理 Socket 连接
- 发送 JSON Action
- 接收 JSON ActionResult
- 处理请求与响应的 `Completer` 映射
- 处理桌面端 crash / shutdown 场景

为什么它重要：

- 桌面端 Core 问题很多都不是 Go 本身的问题，而是这里的进程与通信链路问题。

适合什么时候看：

- 桌面端 Core 起不来
- 请求发送后没返回
- Core 意外退出
- Socket 断开后状态不一致

## 5.5 Android / 动态库实现

### `lib/core/lib.dart`

作用：

- 从命名和 `controller.dart` 的使用方式看，这是 Android 或动态库路径下的 Core 实现。

说明：

- 本文没有深入展开文件内容，这里是基于调用关系给出的定位建议。

适合什么时候看：

- Android 平台 Core 调用异常
- FFI/动态库初始化相关问题

## 5.6 事件总线

### `lib/core/event.dart`

它定义了：

- `CoreEventListener`
- `CoreEventManager`

负责把 Core 回传事件广播给监听者，当前关键事件包括：

- `log`
- `delay`
- `request`
- `loaded`
- `crash`

为什么它重要：

- 这就是 Core 事件从底层回到 Flutter 状态层的桥。
- `CoreManager` 正是监听这套事件并回写 Provider。

适合什么时候看：

- 日志页没更新
- 延迟结果没刷新
- 请求跟踪没数据
- Core crash 没反应

## 5.7 `core` 层开发建议

如果你要新增一个 Core 能力，推荐顺序是：

1. 在 Go Core 定义动作和实现
2. 在 `lib/core/interface.dart` 补协议方法
3. 在 `lib/core/controller.dart` 暴露为可调用接口
4. 如有桌面特殊逻辑，再检查 `lib/core/service.dart`
5. 最后接入 `providers` / `views`

这样最稳，不容易改漏。

## 6. 三层联动时怎么定位

## 6.1 改一个设置项

通常路径：

- `lib/views/config/...`
- `lib/providers/config.dart`
- `lib/models/config.dart`
- 必要时 `lib/controller.dart`
- 如果会影响内核，再看 `lib/core/controller.dart`

## 6.2 改代理组显示

通常路径：

- `lib/views/proxies/...`
- `lib/providers/state.dart`
- `lib/core/controller.dart`
- `lib/manager/core_manager.dart`

## 6.3 改 Profile 管理

通常路径：

- `lib/views/profiles/...`
- `lib/providers/database.dart`
- `lib/database/`
- `lib/controller.dart`

## 6.4 改请求/日志/连接页

通常路径：

- `lib/views/connection/...` 或 `lib/views/logs.dart`
- `lib/core/event.dart`
- `lib/manager/core_manager.dart`
- `lib/providers/state.dart`

## 7. 建议你最先熟悉的文件

如果你现在要开始实际开发，最值得优先打开的 10 个文件是：

1. `lib/views/views.dart`
2. `lib/views/dashboard/dashboard.dart`
3. `lib/views/profiles/profiles.dart`
4. `lib/views/proxies/proxies.dart`
5. `lib/views/config/config.dart`
6. `lib/providers/config.dart`
7. `lib/providers/state.dart`
8. `lib/providers/database.dart`
9. `lib/core/controller.dart`
10. `lib/core/interface.dart`

这 10 个文件基本能帮你建立“页面、状态、Core”三层对应关系。

## 8. 一句话定位法

你以后遇到问题时，可以先用这句简单规则判断入口：

- 看起来像页面问题，先进 `views`
- 看起来像状态问题，先进 `providers`
- 看起来像调用底层没生效，先进 `core`

