# 漫画 · 第 9 话 团队协作：规范与 Code Review

> 对应正文：[09-team-collaboration.md](../09-team-collaboration.md) ｜ 原画：[EP.09-team-collaboration.svg](./EP.09-team-collaboration.svg)

## 登场角色

- **斯威芙蒂**：团队十个人十种代码风格，PR review 只回「LGTM 👍」，bug 照样合进 main。
- **Runtime 长老**：教她用工具强制规范、用清单保真 review。

## 剧情梗概

没有工具约束的规范等于没规范——你写 `func doThing(_ x:Int)->String{`，他写 `func doThing(_ x: Int) -> String {`，review 时没人较真。正确的做法是：SwiftLint 配置进仓库、CI 不过不合并；Code Review 对照五查清单（正确性/可读性/测试/性能/安全）逐项打勾；Git Flow 保证 main 永远可发布。

## 分格解读

### 格1 · 痛点现场

两个典型问题：

- **风格混乱**：没有强制工具，团队代码风格靠「自觉」，结果一个文件里三种缩进、两种命名。
- **Review 走过场**：PR 扔上去没人认真看，回复「LGTM」就合并，bug 混进主干。

根本原因：**规范没有工具化，review 没有清单化**。

### 格2 · 机制登场

- **SwiftLint 强制风格**：`.swiftlint.yml` 配置规则（行长度、命名、强制解包禁用等），提交前本地跑、CI 上跑，不通过不合并。规范当代码管，进仓库全员生效。
- **Swift API 设计准则**：命名清晰（`remove(at:)` 而非 `remove(_:)`）、协议前置（先定义协议再写实现）、错误显式（用 `throws` 而非返回 `nil`/错误码）。
- **Git Flow**：feature 分支开发 → PR → 至少 1 人 approve + CI 通过 → squash merge 到 main。main 分支保护，禁止直推。
- **模块化**：按业务拆成独立 SPM target 或 Framework，编译隔离、依赖单向（上层依赖下层，下层不能反向依赖上层）。

### 格3 · 落地收束

Code Review 五查清单：

1. **查正确性**：边界条件、错误处理、并发安全（Actor/锁）、Optional 处理；
2. **查可读性**：命名语义清晰、函数单一职责、关键逻辑有注释；
3. **查测试**：新逻辑有单测、改动不破坏旧测试、不通过 `// swiftlint:disable` 绕过规则；
4. **查性能**：避免 O(n²) 循环、主线程不做 IO、图片按需解码；
5. **查安全**：无强制解包、无硬编码密钥/URL、机密走 Keychain。

## Runtime 长老划重点

1. 规范靠工具落地：SwiftLint 进 CI，不通过不合并，不靠人盯。
2. Code Review 五查：正确性/可读性/测试/性能/安全，对照清单不走过场。
3. 模块化拆分依赖，feature 分支 + PR 审批，主干永远可发布。

## 自测一题

**问**：为什么不推荐在 PR 里同时做「功能开发 + 重构 + 格式化」三件事？

**答**：因为这会让 review 成本爆炸。一个 500 行的 PR 里混了格式化（300 行空白/缩进变动）+ 重构（150 行移动）+ 新功能（50 行），reviewer 很难定位真正的逻辑变更，很容易漏掉 bug。正确做法：格式化单独提一个 PR（只动空白和风格），重构单独提一个 PR（只移动不改逻辑），新功能单独提一个 PR。每个 PR 职责单一，review 快且准。

## 去实验室

安装 `brew install swiftlint`，在项目根目录创建 `.swiftlint.yml` 并运行 `swiftlint lint`；阅读第 9 章的 Code Review 检查清单模板和模块化拆分方案。

## 下一话预告

团队协作顺了，可金融 App 要合规、医疗 App 要 HIPAA、零售 App 要高并发。第 10 话《企业场景：行业与合规》，看通用工程能力如何映射到具体行业。
