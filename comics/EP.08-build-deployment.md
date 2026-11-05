# 漫画 · 第 8 话 构建部署：签名与 CI/CD

> 对应正文：[08-build-deployment.md](../08-build-deployment.md) ｜ 原画：[EP.08-build-deployment.svg](./EP.08-build-deployment.svg)

## 登场角色

- **斯威芙蒂**：发版前夜证书过期、描述文件不匹配，全公司等她重签。
- **Runtime 长老**：教她用 fastlane 把签名、打包、上架全自动化。

## 剧情梗概

iOS 签名体系是新手最大的坎：证书（Certificate）证明你是谁，描述文件（Provisioning Profile）授权哪些设备和能力，Entitlements 声明具体权限。手动管理在多人协作时必然混乱——证书过期、私钥丢失、描述文件不匹配。解法是用 fastlane 的 `match` 加密存储证书到私有 Git 仓库，`gym` 打包，`pilot` 上传 TestFlight，配合 CI/CD 流水线实现全链路无人值守。

## 分格解读

### 格1 · 痛点现场

`No matching provisioning profile found`、`Certificate has expired`——这两类错误在手动管理签名时太常见。发版前夜证书过期，团队所有人停手等重签；换了新电脑，私钥没导出，编不过。**签名不是玄学，但手动管理必然翻车**。

### 格2 · 机制登场

签名体系三件套：

- **证书（Certificate）**：基于 Apple 颁发的 CSR 生成，证明开发者/团队身份。分为 Development（调试）和 Distribution（发布）。
- **描述文件（Provisioning Profile）**：把证书、App ID、设备列表、Entitlements 打包在一起，告诉系统「这个 App 可以在这些设备上用这些权限运行」。
- **Entitlements**：声明 App 需要的能力（推送、iCloud、App Group 等），必须与描述文件中的一致。

fastlane 三件套：

- **match**：把证书和描述文件加密存到私有 Git 仓库，所有开发者/CI 机器一键同步，告别「私钥在谁电脑上」的混乱。
- **gym**：封装 `xcodebuild archive`，自动处理签名、导出 ipa。
- **pilot**：上传 ipa 到 TestFlight / App Store Connect。

### 格3 · 落地收束

CI/CD 流水线四步：

1. **PR 阶段**：SwiftLint 检查风格 + 单元测试全量通过，不通过不能合并；
2. **合并 main**：触发 archive，`fastlane match` 拉证书，`gym` 出 ipa；
3. **发布 TestFlight**：`pilot` 上传，自动分发给内部测试组；
4. **App Store 发布**：审核通过后一键提交。

配置分离用 `.xcconfig` 文件，Debug/Release/Enterprise 各一套，不污染 `project.pbxproj`。

## Runtime 长老划重点

1. 签名三件套：证书证明身份、描述文件授权、Entitlements 声明权限。
2. 用 fastlane match 加密存证书，换机器/换人不再卡签名。
3. CI/CD 流水线：lint→test→archive→上传，全链路无人值守。

## 自测一题

**问**：为什么 iOS App 必须签名？不签名会怎样？

**答**：签名是 Apple 平台的安全基石，目的有二：①防止篡改——App 签名后，任何对二进制的修改都会导致签名校验失败，系统拒绝运行；②身份溯源——证书绑定开发者账号，出了安全问题 Apple 能追溯到责任主体。不签名的 App 无法安装到真机（模拟器不需要签名），也无法上架 App Store。企业内部分发用企业证书，同样需要签名。

## 去实验室

查看 `examples/fastlane/` 下的 Fastfile / Matchfile / Gymfile，理解 fastlane 配置；阅读第 8 章关于 `xcodebuild archive -exportArchive` 的完整命令和 `.xcconfig` 写法。

## 下一话预告

流水线跑通了，可团队代码风格五花八门、PR review 走过场。第 9 话《团队协作：规范与 Code Review》，建一套让代码质量不掉线的协作规范。
