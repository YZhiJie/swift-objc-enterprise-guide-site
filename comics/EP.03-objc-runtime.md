# 漫画 · 第 3 话 ObjC 运行时：消息机制与转发

> 对应正文：[03-objc-runtime.md](../03-objc-runtime.md) ｜ 原画：[EP.03-objc-runtime.svg](./EP.03-objc-runtime.svg)

## 登场角色

- **斯威芙蒂**：写了 `[obj performSelector:@selector(foo)]`，方法名拼错，App 直接 `unrecognized selector` 闪退。
- **Runtime 长老**：带她看 `objc_msgSend` 的查找流程和消息转发三步兜底。

## 剧情梗概

ObjC 的方法调用不是函数调用，而是「给对象发消息」。编译器只负责把 `[obj foo]` 翻译成 `objc_msgSend(obj, @selector(foo))`，方法存不存在要到运行时才查。查不到也不会立刻崩——运行时会依次尝试三步消息转发，只有三步都不处理才抛 `unrecognized selector`。这套机制正是 KVO、关联对象、方法欺骗的底层基础。

## 分格解读

### 格1 · 痛点现场

`unrecognized selector sent to instance` 是 ObjC 最经典的崩溃之一。根源在于 ObjC 的动态性：方法调用在编译期不做存在性检查，selector 只是一个字符串，到运行时才去对象的类上查找方法实现。写错方法名、或者对象实际类型和声明不符，都会触发。

### 格2 · 机制登场

`objc_msgSend` 的查找链路：

1. 先查对象的方法缓存（`objc_msgSend` 用汇编手写，极快）；
2. 缓存未命中，查类的方法列表；
3. 再沿父类链向上查；
4. 全程没找到，进入**消息转发**。

消息转发三步：

- **第一步 `resolveInstanceMethod:`**：给类一个机会动态添加方法实现（`class_addMethod`）；
- **第二步 `forwardingTargetForSelector:`**：返回另一个对象，让它去响应这个 selector（最轻量）；
- **第三步 `methodSignatureForSelector:` + `forwardInvocation:`**：拿到完整的 `NSInvocation`，可以任意修改参数、目标，再调用。

三步全部返回默认值或 nil，才会走到 `doesNotRecognizeSelector:` 抛异常。

### 格3 · 落地收束

- 想让某个未知 selector 不崩，最轻量的做法是重写 `forwardingTargetForSelector:` 返回备用对象；
- 想实现「动态属性」「自动 NSCoding」这类黑魔法，用 `resolveInstanceMethod:` + `class_addMethod` 在运行时补方法；
- KVO 的本质就是 Runtime 在运行时动态子类化，重写 setter 插入 `willChangeValueForKey:` / `didChangeValueForKey:`。

## Runtime 长老划重点

1. ObjC 方法调用即发消息，编译期不检查，运行时才查方法列表。
2. 找不到方法走三步转发：动态添加→转对象→完整 Invocation。
3. KVO、关联对象、方法欺骗全靠 Runtime API，混编时读懂它们不踩坑。

## 自测一题

**问**：为什么给 `nil` 发消息在 ObjC 里不会崩溃，而 Swift 强制解包 nil 会？

**答**：ObjC 的 `objc_msgSend` 第一行就判断 `if (obj == nil) return;`，给 nil 发消息直接返回 0/NULL/nil，不做任何查找。这是 ObjC 的语言设计。而 Swift 的 `!` 强制解包在运行时会检查 Optional 是否为 `.none`，是 nil 就主动触发 `fatalError`。两者的「空」哲学完全不同：ObjC 宽容静默，Swift 明确失败。

## 去实验室

查看 `examples/Sources/ObjCUtils/ObjCUtils.m` 中的 nullability 注解与方法声明，再阅读第 3 章关于 `class_addMethod` 和 `method_exchangeImplementations` 的示例，在 Xcode 里新建一个 ObjC 类实验消息转发第二步。

## 下一话预告

Runtime 黑魔法看完了，可新写的 Swift 代码要跟老 ObjC 类互相调用，桥接头怎么配、`@objc` 怎么加？第 4 话《混编：桥接头与渐进迁移》，打通双语言壁垒。
