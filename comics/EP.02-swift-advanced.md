# 漫画 · 第 2 话 Swift 高级：结构化并发与 Actor

> 对应正文：[02-swift-advanced.md](../02-swift-advanced.md) ｜ 原画：[EP.02-swift-advanced.svg](./EP.02-swift-advanced.svg)

## 登场角色

- **斯威芙蒂**：用 GCD 开十个线程给计数器加一，结果总数偶尔是 998，Thread Sanitizer 一片红。
- **Runtime 长老**：告诉她 Swift 6 把并发安全也关进了类型系统。

## 剧情梗概

GCD 只管把任务派发到队列，共享状态的安全靠程序员自己加锁——加漏一处就是线上数据错乱。Swift 6 的答案是 `async/await` 结构化并发加 `Actor` 隔离：任务有父子层级、取消自动传播；可变状态被 Actor 包住，外部访问必须 `await`，数据竞争在编译期就被拦住。

## 分格解读

### 格1 · 痛点现场

`DispatchQueue.global().async { self.count += 1 }` 开十个并发，`count` 偶尔停在 998。Thread Sanitizer 报 data race：`+=` 不是原子操作，读-改-写三步之间被另一个线程打断，丢了一次自增。GCD 不提供数据隔离，全靠 `NSLock`/串行队列兜底，漏一处就出事。

### 格2 · 机制登场

Swift 并发的两块基石：

- **async/await 结构化并发**：`async let` 启动并行子任务，父任务 `await` 时自动等待所有子任务；`Task` 形成任务树，父任务取消会递归传播给所有子任务，不需要手动维护取消标志。
- **Actor**：`actor` 关键字声明的类型，其可变状态只允许在 Actor 内部串行访问；外部调用其方法或属性必须 `await`，由运行时排队。这等于把「加锁」这件事由语言自动完成。

Swift 6 严格并发模式下，跨线程传递的值必须满足 `Sendable` 协议，编译器在编译期就能发现潜在的数据竞争。

### 格3 · 落地收束

- 用 `actor Counter { private var count; func increment() {} }` 封装计数器，外部 `await counter.increment()`，天然线程安全；
- 用 `async let users = fetchUsers(); async let posts = fetchPosts()` 并行发起请求，`let (u, p) = try await (users, posts)` 一次取回；
- 长任务用 `Task {}` 包装，需要取消时调 `task.cancel()`，子任务里用 `Task.isCancelled` 检查或 `try Task.checkCancellation()` 响应。

## Runtime 长老划重点

1. 用 Actor 封装可变状态，外部访问一律 await，编译期杜绝数据竞争。
2. async let 并行子任务、Task 树自动传播取消，告别手写回调地狱。
3. Swift 6 严格并发默认开启，Sendable 约束把线程安全写进类型。

## 自测一题

**问**：`actor BankAccount { var balance = 0; func deposit(_ a: Int) { balance += a } }`，两个 Task 同时调 `await account.deposit(100)`，balance 会出错吗？

**答**：不会。Actor 保证同一时刻只有一个任务能执行其方法，`deposit` 内部对 `balance` 的读写是串行化的，不存在数据竞争。如果把 `actor` 改成 `class`，就需要自己加锁；这正是 Actor 的价值——把并发安全从「程序员纪律」变成「编译器保证」。

## 去实验室

运行 `swift run ActorBank`，对比「无隔离 class 账户」与「Actor 账户」在 1000 次并发存取后的余额差异；再跑 `swift run AsyncBasics` 体验 `async let` 并行与 `Task` 取消。

## 下一话预告

并发安全搞定了，可老项目里全是 `[obj doSomething:]`，这些方括号到底在干嘛？第 3 话《ObjC 运行时：消息机制与转发》，拆开 `objc_msgSend` 的黑盒。
