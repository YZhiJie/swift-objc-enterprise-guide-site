# 漫画 · 第 14 话 系统触点：Widget 与 App Intents

> 对应正文：[14-system-touchpoints.md](../14-system-touchpoints.md) ｜ 原画：[EP.14-system-touchpoints.svg](./EP.14-system-touchpoints.svg)

## 登场角色

- **斯威芙蒂**：Widget 半天不刷新、喊 Siri 没反应、推送通知显示的是密文。App 孤立在系统之外，用户不打开就触达不了。
- **Runtime 长老**：教她用 WidgetKit、App Intents、通知扩展三大触点把 App 能力延伸到系统层面。

## 剧情梗概

App 的价值不只在打开时，更在用户不打开时也能提供服务。三大系统触点：①WidgetKit 用时间线机制在锁屏/桌面展示信息，有系统刷新预算；②App Intents 让 Siri 和快捷指令能调用 App 能力；③通知 Service 扩展在送达前解密切文，既保护隐私又展示可读内容。主 App 和扩展通过 App Group 共享数据。

## 分格解读

### 格1 · 痛点现场

- **Widget 不刷新**：以为像 App 一样想刷就刷，结果系统有刷新预算（每小时几次），频繁请求被节流；
- **Siri 无响应**：没有定义 App Intents，Siri 不知道你的 App 能做什么；
- **通知是密文**：服务端推送加密内容，直接显示用户看不懂，又不想在 App 内解密（延迟）。

### 格2 · 机制登场

三大系统触点：

- **WidgetKit**：不是实时刷新，而是用 `TimelineProvider` 预生成一批时间点的视图快照（`TimelineEntry`），系统按时间展示。刷新策略 `.atEnd`（时间线结束再请求下一批，最省预算）、`.after(date)`（指定时间刷新）、`.never`（静态不刷新）。
- **App Intents**：定义 `AppIntent` 协议的结构体，声明参数和执行方法，系统自动接入 Siri、快捷指令、聚焦搜索、Action Button。三角色：AppIntent（做什么）、AppEntity（操作对象）、AppEnum（枚举参数）。
- **通知扩展**：`UNNotificationServiceExtension` 在通知送达用户前拦截，解密 payload 中的密文，替换为明文再展示。30 秒超时，超时走原始内容。
- **App Group**：主 App 和扩展是独立进程，通过 `group.<bundle-id>` 共享 UserDefaults、文件容器、Keychain 项目。

### 格3 · 落地收束

1. **Widget 时间线策略**：用 `.atEnd` 让系统在时间线末尾再请求下一批，避免频繁刷新耗尽预算；
2. **深链打通**：`widgetURL` 或 `Link(destination:)` 让点击 Widget 跳转到 App 对应页面；
3. **通知解密**：Service Extension 用共享的密钥（App Group Keychain）解密切文，展示明文标题。

## Runtime 长老划重点

1. Widget 用 Timeline 预生成快照，刷新有系统预算，不要频繁请求。
2. App Intents 让 Siri/快捷指令调用 App 能力，App Group 共享数据给扩展。
3. 通知 Service 扩展在送达前解密切文，保护隐私同时展示可读内容。

## 自测一题

**问**：Widget 为什么不能像 App 一样实时刷新数据？

**答**：因为 Widget 运行在独立进程，系统为了省电和性能，对 Widget 的刷新有严格预算（每小时几次，具体由系统根据用户使用频率动态调整）。如果允许实时刷新，等于每个 Widget 都在后台常驻跑网络，电池和性能扛不住。所以 WidgetKit 设计成「时间线快照」模式：你一次性提供未来一段时间的数据快照，系统按时间展示，到时间线末尾再回调你请求下一批。要实时更新只能用 `WidgetCenter.shared.reloadAllTimelines()`，但要省着用。

## 去实验室

运行 `swift run SystemTouchpoints`，查看 `TimelinePolicy.swift`（刷新策略决策）、`WidgetDeepLink.swift`（深链往返）和 `NotificationEnvelope.swift`（通知解密降级）的实现；阅读第 14 章的 App Intents 三角色和 App Group 配置。

## 下一话预告

系统触点全覆盖了，可上线后崩溃率升高、用户反馈卡顿、灰度发布不知道该不该全量。第 15 话《运营闭环：可观测性与灰度》，让数据驱动运营决策。
