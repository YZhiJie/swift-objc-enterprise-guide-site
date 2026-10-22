# 漫画 · 第 6 话 性能与安全：Instruments 与 Keychain

> 对应正文：[06-performance-security.md](../06-performance-security.md) ｜ 原画：[EP.06-performance-security.svg](./EP.06-performance-security.svg)

## 登场角色

- **斯威芙蒂**：App 启动 3.2 秒、滑动卡顿，token 还明文存在 UserDefaults 里。
- **Runtime 长老**：教她用 Instruments 量出瓶颈、用 Keychain 守住机密。

## 剧情梗概

性能问题不能靠感觉优化——你以为是图片大，实际是 `+load` 里同步初始化了五个 SDK。安全问题不能靠侥幸——token 存 UserDefaults 等于把钥匙放在门垫下。正确的做法是：用 Instruments（Time Profiler / Allocations / Leaks）先测量再定位；机密一律走 Keychain；网络层开 ATS + 证书绑定防中间人。

## 分格解读

### 格1 · 痛点现场

两个典型坑：

- **性能**：启动慢、列表滑动掉帧，但不知道卡在哪一行。凭感觉优化往往改了一堆无关代码，真正的瓶颈（主线程 IO、同步初始化、图片解码）纹丝不动。
- **安全**：把登录 token、API 密钥写进 `UserDefaults`。UserDefaults 本质是 plist 文件，越狱设备或备份可直接读取，等于明文。

### 格2 · 机制登场

- **Instruments**：Xcode 自带的性能分析套件。`Time Profiler` 看 CPU 耗时栈，`Allocations` 看内存分配，`Leaks` 查内存泄漏，`Energy Log` 看耗电。**先测再改**是铁律。
- **Keychain**：系统级加密存储，数据被 Apple 的 Secure Enclave 保护，App 卸载后仍保留（同开发者账号下可共享）。用 `kSecClassGenericPassword` 存储普通机密，`kSecAttrAccessibleWhenUnlocked` 控制访问时机。
- **ATS（App Transport Security）**：iOS 9+ 默认强制 HTTPS，阻止明文 HTTP。`Certificate Pinning`（证书绑定）把服务器证书的公钥哈希内置到 App，防止中间人用伪造证书窃听。

### 格3 · 落地收束

启动优化三砍：

1. **砍启动任务**：把非首屏必需的 SDK 初始化从 `didFinishLaunching` 移到首屏渲染后；
2. **砍内存峰值**：图片按需解码（`ImageIO` 按屏幕尺寸解码）、列表行复用、大循环包 `autoreleasepool`；
3. **机密进 Keychain**：封装一个 `KeychainHelper`，统一读写 token / 密钥，禁止再用 UserDefaults。

## Runtime 长老划重点

1. 先测量再优化，Instruments 定位瓶颈，禁止凭感觉改代码。
2. 机密（token/密钥）只存 Keychain，UserDefaults 只存非敏感配置。
3. ATS 强制 HTTPS + Certificate Pinning，防止中间人窃听流量。

## 自测一题

**问**：为什么 Keychain 比 UserDefaults 更安全？App 卸载后 Keychain 数据还在吗？

**答**：Keychain 数据由系统加密存储（底层是 AES 加密的 SQLite，密钥在 Secure Enclave），App 只能通过 Security 框架 API 访问自己 Keychain Group 里的数据，无法直接读文件。UserDefaults 是明文 plist。Keychain 数据在 App 卸载后默认保留（同 Team ID 下重装可恢复），这是有意设计——用户重装 App 不需要重新登录。如需卸载即清除，可在 `applicationWillTerminate` 时主动删除。

## 去实验室

运行 `swift run EnterprisePatterns`，查看 `SecureStore.swift` 的 Keychain 封装和 `PinningURLSession.swift` 的证书绑定实现；用 Xcode Open Developer Tool → Instruments 跑一次 Time Profiler。

## 下一话预告

性能和安全加固了，可上线前怎么保证不把 bug 带出去？第 7 话《测试：XCTest 与质量门禁》，建一套能自动拦截坏代码的测试体系。
