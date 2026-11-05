# 漫画 · 第 11 话 iOS 架构：MVC/MVVM/TCA 选型

> 对应正文：[11-ios-architecture.md](../11-ios-architecture.md) ｜ 原画：[EP.11-ios-architecture.svg](./EP.11-ios-architecture.svg)

## 登场角色

- **斯威芙蒂**：一个 UserViewController 写了 1500 行，业务逻辑、网络、定位、UI 全混在一起，改一处崩三处。
- **Runtime 长老**：教她按「状态复杂度」选架构，不要盲目追新。

## 剧情梗概

MVC 本没错，错在把所有东西都塞进 Controller，变成 Massive View Controller。架构选型的核心维度是**状态复杂度**：简单静态页用 MVC 就够，中等业务用 MVVM + @Observable，多源状态/撤销重做/状态回放才上 TCA。主力方案是 MVVM，状态集中在 ViewModel，View 只做渲染。

## 分格解读

### 格1 · 痛点现场

一个 VC 同时实现 `UITableViewDataSource`、`URLSessionDelegate`、`CLLocationManagerDelegate`，业务逻辑、网络请求、定位回调、UI 更新全在一个文件里。改一个网络回调，列表刷新逻辑也跟着动；加一个定位权限判断，三处 if 要同步改。**职责不分离，复杂度指数级增长**。

### 格2 · 机制登场

三轴对比：

| 架构 | 适用场景 | 优点 | 缺点 |
| --- | --- | --- | --- |
| **MVC** | 简单页面、原型 | 上手快、代码少 | Controller 易胖，难测试 |
| **MVVM** | 主流业务页面 | 状态集中、可测试、与 SwiftUI 天然契合 | 需要约定数据流 |
| **TCA** | 复杂状态机、多源状态 | 单向数据流、可回放、可撤销重做 | 学习曲线陡、样板代码多 |

选型决策树：**状态只有一个来源 → MVC；状态来自多个输入但可集中管理 → MVVM；状态需要回溯/撤销/多组件协调 → TCA**。

### 格3 · 落地收束

- **简单页面（设置页、静态展示）**：直接 MVC，不强行套架构，过度设计也是坏味道；
- **业务页面（列表+详情+网络）**：MVVM，`@Observable class ViewModel` 持有状态和业务方法，View 通过 `@State` 持有 ViewModel 实例，状态变化自动刷新；
- **复杂场景（编辑器、工作流、多步表单）**：TCA，State/Action/Reducer/Environment 四件套，状态可回放可测试。

## Runtime 长老划重点

1. MVC 易胖、MVVM 平衡、TCA 严谨，按状态复杂度选型，不盲目追新。
2. 主力 MVVM + @Observable，状态集中在 ViewModel，View 只做渲染。
3. 架构目标是「可测、可维护」，不是「套名词」，简单页面别强行上 TCA。

## 自测一题

**问**：MVVM 里 ViewModel 应该持有 View 的引用吗？为什么？

**答**：不应该。MVVM 的核心是「ViewModel 不依赖 View」，这样 ViewModel 可以脱离 UI 独立测试。数据流是单向的：View → ViewModel（用户操作调方法）→ ViewModel 更新状态 → View 观察状态自动刷新。如果 ViewModel 持有 View 引用，就形成了双向依赖，测试时必须 mock View，且容易出现循环引用。SwiftUI 里 View 通过 `@State`/`@StateObject` 持有 ViewModel，ViewModel 完全不知道 View 的存在。

## 去实验室

运行 `swift run ArchitecturePatterns`，对比同一需求的 MVC（反例）、MVVM（@Observable 状态机）、迷你 TCA 三种实现；阅读第 11 章的选型决策树和类型安全导航方案。

## 下一话预告

架构稳了，该给 App 加点「智能」了——图片分类、文字识别、本地推理。第 12 话《端上智能：Core ML 与设备侧 AI》，看模型怎么跑在用户手机上。
