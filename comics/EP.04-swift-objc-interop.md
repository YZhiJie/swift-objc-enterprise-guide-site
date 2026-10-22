# 漫画 · 第 4 话 混编：桥接头与渐进迁移

> 对应正文：[04-swift-objc-interop.md](../04-swift-objc-interop.md) ｜ 原画：[EP.04-swift-objc-interop.svg](./EP.04-swift-objc-interop.svg)

## 登场角色

- **斯威芙蒂**：在 Swift 文件里写 `let vc = OldObjCViewController()`，编译器报 `Cannot find in scope`；反过来 ObjC 也找不到 Swift 的方法。
- **Runtime 长老**：教她配两座桥——桥接头和 `@objc`。

## 剧情梗概

Swift 和 ObjC 各有一套符号表，默认互不可见。要让它们互相调用，需要两座桥：**桥接头**（Bridging Header）让 Swift 看见 ObjC，**`@objc` 标记**让 ObjC 看见 Swift。再配合 nullability 注解和 `NS_SWIFT_NAME`，就能把 ObjC 的指针精确映射成 Swift 的 Optional，消除隐式解包的雷区。

## 分格解读

### 格1 · 痛点现场

混合工程里最常见的两类报错：Swift 侧 `Cannot find 'XXX' in scope`（桥接头没配或没 import 对应头），ObjC 侧 `No visible @interface declares the selector`（Swift 类没加 `@objc` 或没继承 `NSObject`）。两种语言各有独立的符号空间，中间必须有显式的桥。

### 格2 · 机制登场

两座桥的工作方式：

- **ObjC → Swift（桥接头）**：Xcode 自动生成或手动创建 `*-Bridging-Header.h`，在其中 `#import` 需要暴露给 Swift 的 ObjC 头文件。导入后的类、协议、category 在 Swift 中直接可用。
- **Swift → ObjC（`@objc` + 自动生成头）**：继承 `NSObject` 并标记 `@objc` 的类/方法，编译器会自动生成 `<ProductName>-Swift.h` 头，ObjC 文件 import 它即可调用 Swift 代码。
- **nullability 注解**：`nonnull`/`nullable`/`null_unspecified` 把 ObjC 指针映射成 Swift 的非 Optional/Optional/隐式解包 Optional。企业规范要求公共头全部加注解，杜绝 `String!`。
- **`NS_SWIFT_NAME` / `NS_REFINED_FOR_SWIFT`**：自定义 Swift 侧的命名，或对 Swift 侧重新封装更地道的 API。

### 格3 · 落地收束

渐进迁移三步走，禁止「大爆炸重写」：

1. **先补注解，不删代码**：给 ObjC 公共头加 nullability，Swift 侧立刻从 `String!` 变成 `String?`，安全收益零风险；
2. **新功能用 Swift 写**：通过 `@objc` 暴露给老 ObjC 调用，双轨共存；
3. **逐个模块替换**：每次只改一个模块，跑通测试再换下一个，最后删空 ObjC 目标。

## Runtime 长老划重点

1. ObjC→Swift 靠桥接头，Swift→ObjC 靠 @objc，两座桥缺一不可。
2. nullability 注解把 ObjC 指针精确映射成 Swift Optional，消除隐式解包。
3. 迁移策略：先补注解→新功能 Swift 写→逐模块替换，禁止大爆炸重写。

## 自测一题

**问**：一个 Swift 类 `class Utils { static func run() {} }`，ObjC 里能调用 `[Utils run]` 吗？需要改什么？

**答**：不能。首先 `Utils` 没有继承 `NSObject`，不会生成 ObjC 运行时类；其次 `run()` 没加 `@objc`。正确写法：`@objc class Utils: NSObject { @objc static func run() {} }`。ObjC 文件 `#import "<Product>-Swift.h"` 后即可 `[Utils run]`。注意 struct、enum、泛型默认不暴露给 ObjC。

## 去实验室

运行 `swift run InteropDemo` 查看 Swift 调用 ObjC 的桥接效果；打开 `GuideLab/GuideLab/SupportingFiles/GuideLab-Bridging-Header.h` 看真实工程的桥接头配置。

## 下一话预告

语言层打通了，该写 UI 了。UIKit 命令式和 SwiftUI 声明式到底怎么选？第 5 话《框架：UIKit 与 SwiftUI 双轨》，看两套 UI 框架的分工与共存。
