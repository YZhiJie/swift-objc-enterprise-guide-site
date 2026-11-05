# 漫画 · 第 5 话 Apple 框架：UIKit 与 SwiftUI 双轨

> 对应正文：[05-apple-frameworks.md](../05-apple-frameworks.md) ｜ 原画：[EP.05-apple-frameworks.svg](./EP.05-apple-frameworks.svg)

## 登场角色

- **斯威芙蒂**：用 UIKit 写页面，改了 model 忘了刷 label，界面状态对不上。
- **Runtime 长老**：教她用 SwiftUI 声明式把「同步」这件事交给框架。

## 剧情梗概

UIKit 是命令式——你一行行告诉视图怎么摆、怎么改，状态变了得手动刷每一处 UI，漏一个就是 bug。SwiftUI 是声明式——你只描述「状态是什么、界面该长什么样」，框架自动 diff 并刷新。两套框架不是二选一，而是双轨共存：新页面用 SwiftUI，老页面留 UIKit，中间靠 `UIHostingController` 和 `UIViewRepresentable` 互嵌。

## 分格解读

### 格1 · 痛点现场

UIKit 里 `label.text = model.title`、`button.isEnabled = model.canSubmit`、`tableView.reloadData()` 散落在各处。一旦 model 变了但忘了调某行刷新，界面就出现「数据已变、UI 还停在旧状态」的不一致。这就是命令式的代价：状态同步全靠程序员盯，复杂度随视图数量线性增长。

### 格2 · 机制登场

两套范式的本质区别：

- **SwiftUI 声明式**：用 `@State` / `@Observable` 标记状态源，视图是状态的纯函数 `body = f(state)`。状态一变，框架自动重新计算 body、diff 出最小变更、只刷新受影响的视图。
- **UIKit 命令式**：直接操作视图对象（addSubview、setTitle、reloadData），你精确控制每一步，但也要对每一步的正确性负责。

选型不是非此即彼：列表、表单、详情页这类标准界面 SwiftUI 效率最高；自定义绘制、复杂手势、极致性能优化仍用 UIKit + Core Animation。

### 格3 · 落地收束

双轨共存的三种桥接：

1. **新页面优先 SwiftUI**：用 `@Observable` 驱动状态，`body` 描述界面；
2. **UIKit 嵌 SwiftUI**：`UIHostingController(rootView: MySwiftUIView())` 把 SwiftUI 视图塞进 UIKit 流程；
3. **SwiftUI 嵌 UIKit**：实现 `UIViewRepresentable` / `UIViewControllerRepresentable`，把 `MKMapView`、`WKWebView` 等 UIKit 控件包装进 SwiftUI。

企业规范：纯 Swift 新工程以 SwiftUI 为主，遗留工程按页面粒度渐进替换。

## Runtime 长老划重点

1. SwiftUI 声明式描述「要什么」，状态驱动 UI 自动刷新，杜绝手动同步遗漏。
2. UIKit 仍负责复杂自定义绘制与细粒度性能控制，两套框架长期共存。
3. 互嵌用 UIHostingController / UIViewRepresentable，新老页面平滑过渡。

## 自测一题

**问**：SwiftUI 里 `@State` 和 `@Observable` 有什么区别？该选哪个？

**答**：`@State` 是 SwiftUI 原生的属性包装器，值类型专用，存储由 SwiftUI 框架管理；`@Observable` 是 iOS 17+ Observation 框架的宏，可用于 class，且只有被视图实际读取的属性变化才触发刷新（精准依赖追踪）。新项目优先 `@Observable`（iOS 17+），兼容性要求高或值类型局部状态用 `@State`。

## 去实验室

打开 `GuideLab/GuideLab/Labs/` 目录，每个 Lab 都是一个 SwiftUI 页面；运行 `GuideLab` target 体验 SwiftUI 实时预览。再阅读第 5 章关于 `UIViewRepresentable` 包装 `WKWebView` 的示例。

## 下一话预告

界面跑起来了，可启动要 3 秒、滑动掉帧、Keychain 存的 token 被明文读走。第 6 话《性能与安全：Instruments 与 Keychain》，给 App 做体检和加固。
