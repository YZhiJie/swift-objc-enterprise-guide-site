# 漫画 · 第 1 话 Swift 基础：可选类型与值引用

> 对应正文：[01-swift-basics.md](../01-swift-basics.md) ｜ 原画：[EP.01-swift-basics.svg](./EP.01-swift-basics.svg)

## 登场角色

- **斯威芙蒂**：Swift 鸟学徒，本话主角。从字典取值直接 `.count`，被 `Unexpectedly found nil` 当场劝退。
- **Runtime 长老**：ObjC 运行时老法师，只在知识条出镜，负责带走三句结论。

## 剧情梗概

一行 `dict["name"].count` 把 App 干崩。斯威芙蒂不服——在 ObjC 里给 nil 发消息不是没事吗？长老领她看 Swift 的底牌：`Optional<T>` 把「可能为空」直接写进了类型签名，编译器逼你每条路都走到。再配合值类型与引用类型的二分，Swift 用类型系统替你守住了空指针这道老坑。

## 分格解读

### 格1 · 痛点现场

终端红屏：`Fatal error: Unexpectedly found nil`。字典里没有 `"name"` 这个 key，`dict["name"]` 返回的是 `nil`，直接 `.count` 等于强制解包一个空盒——Swift 二话不说就炸。这不是 bug：**Swift 把「无值」建模成了类型的一部分，你必须显式处理，不能靠运气**。

### 格2 · 机制登场

两个核心机制：

- **Optional**：`String?` 等价于 `Optional<String>`，取值要么是 `.some(String)`，要么是 `.none`。字典下标、可失败初始化器都返回 Optional，编译器不允许你跳过空检查。
- **值类型 vs 引用类型**：`struct` 赋值时拷贝（写时复制 COW），`class` 赋值时共享指针。选 struct 还是 class，先问自己一句「这个数据要不要被多个地方共享修改」。

可选链 `?.` 是 Optional 的好朋友：链上任一节点为 nil，整条表达式短路返回 nil，不会崩。

### 格3 · 落地收束

安全解包三式，生产代码里禁用强制解包 `!`：

1. **`if let`**：有值才进入分支，作用域内保证非空；
2. **`guard let ... else { return }`**：失败早退出，主流程不缩进嵌套；
3. **`?? 默认值`**：nil 时给兜底值，一行表达式搞定。

类型选型口诀：**默认 struct，需要共享身份或引用语义时才用 class**。

## Runtime 长老划重点

1. Optional 把「可能为空」写进类型，编译器替你守住空指针。
2. 安全解包三式：if let / guard let / ??，生产禁用强制 `!`。
3. 值类型用 struct 拷贝、引用类型用 class 共享，选型先问「要不要共享」。

## 自测一题

**问**：`let arr: [Int] = []; let first = arr.first; print(first + 1)` 能编译通过吗？为什么？

**答**：编译不通过。`arr.first` 返回 `Int?`（因为数组可能为空），`Optional<Int>` 不能直接和 `Int` 相加。正确写法是 `print((first ?? 0) + 1)` 或用 `if let first = arr.first { print(first + 1) }`。这正是 Optional 的设计意图——把「数组为空」这种边界条件从运行时 crash 提前到编译期。

## 去实验室

打开 `examples/Sources/NullSafety/main.swift`，把里面所有强制解包 `!` 改写成安全解包三式，运行 `swift run NullSafety` 观察输出差异。再去 `ValueTypes` target 对比 struct 与 class 在赋值后的行为。

## 下一话预告

Optional 守住了空指针，可一段网络回调里同时改 `self.count` 和 `self.array`，数据 race 还是防不住。第 2 话《Swift 高级：结构化并发与 Actor》，看 Swift 6 怎么用 async/await 和 Actor 把并发安全也关进类型系统。
