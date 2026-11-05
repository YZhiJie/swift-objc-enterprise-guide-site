# Swift & ObjC 学园漫画支部 · 风格规范「银翼工坊 Silverwing Workshop」v1.0

> 面向 Apple 企业级开发的漫画教程。每话一张 1200×840 SVG + 一篇讲解 MD，
> 结构固定为：顶部横幅 → 格1 痛点现场 → 格2 机制登场 → 格3 落地收束 → 知识条。
> 校验：`node comics/check-comics.mjs`（0 错误才算通过）。

## 1. 画布与固定布局

| 区域 | 矩形 (x, y, w, h) | 说明 |
| --- | --- | --- |
| 背景 | 0, 0, 1200, 840 | `#f0f4fa`，四角低透明装饰圆 |
| 顶部横幅 | y 6–96 | 双色条 (40,6,120,6 橙 / 40,16,120,6 蓝)、系列名 y=40、大标题 y=80、EP 徽章 (1020,26,140,44) 橙 |
| 分隔线 | y=96 | `#c8d3e8` 2px |
| 格1 痛点现场 | 40, 120, 556, 250 | 圆点编号 `#F05138`，(66,146) r14 |
| 格2 机制登场 | 616, 120, 544, 250 | 圆点编号 `#007AFF` |
| 格3 落地收束 | 40, 390, 1120, 230 | 圆点编号 `#f59e0b` |
| 知识条 | 40, 640, 1120, 152 | 填充 `#fef3c7` 描边 `#f59e0b`；长老 translate(150,744) scale(0.62)；标题 x=230 y=676；三行要点 x=266 y=708/736/764 |
| 页脚 | x=40, y=822 | `.ft`：银翼工坊 Silverwing Workshop · 第 N 话 — 深入阅读 … |

面板统一：`rx=16`，填充 `#ffffff`，描边 `#c8d3e8` 3px；面板内标题 `.pt` 位于圆点右侧 y≈152/422。

## 2. 配色令牌

| 用途 | 色值 |
| --- | --- |
| 页面背景 | `#f0f4fa` |
| 面板白 / 描边 | `#ffffff` / `#c8d3e8` |
| Swift 橙（主色） | `#F05138` |
| Apple 蓝（副色） | `#007AFF` |
| ObjC 紫 | `#7C3AED` |
| 危险红 | `#e5484d` |
| 通过绿 | `#34a853` |
| 高亮琥珀 | `#f59e0b`（知识条文字 `#78350f`） |
| 代码面板 | 填充 `#1e293b` 描边 `#007AFF`，代码字 `#e2e8f0`，强调 `#8fd14f`（绿）/ `#7ab2ff`（蓝）/ `#f0abfc`（紫）/ `#fbbf24`（橙），注释 `#94a3b8` |
| 正文深 / 正文灰 / 弱灰 | `#1e293b` / `#475569` / `#94a3b8` |

## 3. 字符样式（每话 <style> 原样复制）

```css
.t{font-family:'PingFang SC','Hiragino Sans GB','Microsoft YaHei',sans-serif;}
.m{font-family:'SFMono-Regular',Menlo,Consolas,monospace;}
.st{font-size:15px;fill:#64748b;}   /* 系列名 */
.bt{font-size:30px;font-weight:bold;fill:#1e293b;}  /* 话标题 */
.pt{font-size:19px;font-weight:bold;fill:#1e293b;}  /* 面板标题 */
.an{font-size:12.5px;fill:#64748b;} /* 注释行 */
.cd{font-size:11.5px;fill:#e2e8f0;} /* 代码面板文字 */
.kt{font-size:20px;font-weight:bold;fill:#78350f;}  /* 知识条标题 */
.kb{font-size:15px;fill:#78350f;}   /* 知识条要点 */
.ft{font-size:12.5px;fill:#94a3b8;} /* 页脚 */
```

## 4. 角色 defs（每话原样复制后 <use>）

斯威芙蒂（Swift 鸟学徒，原点在脚底，总高约 200）：橙白圆身、尖喙、小翅膀、头顶三根呆毛。
Runtime 长老（知识条专用，原点约在胸口下）：紫袍、圆框眼镜、白胡须、手持苹果法杖。

```svg
<g id="swifty">
  <ellipse cx="0" cy="2" rx="42" ry="7" fill="#1e293b" opacity="0.12"/>
  <path d="M -28 -8 Q -30 -22 -16 -26 L 16 -26 Q 30 -22 28 -8 Z" fill="#F05138"/>
  <ellipse cx="0" cy="-6" rx="32" ry="38" fill="#f8fafc" stroke="#cbd5e1" stroke-width="2.5"/>
  <ellipse cx="0" cy="2" rx="20" ry="22" fill="#ffffff"/>
  <path d="M -8 -8 Q 0 -2 8 -8" stroke="#F05138" stroke-width="2" fill="none" stroke-linecap="round"/>
  <circle cx="-11" cy="-14" r="5.5" fill="#1e293b"/>
  <circle cx="11" cy="-14" r="5.5" fill="#1e293b"/>
  <circle cx="-9" cy="-16" r="1.8" fill="#ffffff"/>
  <circle cx="13" cy="-16" r="1.8" fill="#ffffff"/>
  <path d="M 0 -6 L -4 0 L 4 0 Z" fill="#f97316"/>
  <path d="M -16 -38 L -12 -50 L -8 -38 Z" fill="#F05138"/>
  <path d="M -4 -40 L 0 -54 L 4 -40 Z" fill="#F05138"/>
  <path d="M 8 -38 L 12 -50 L 16 -38 Z" fill="#F05138"/>
  <path d="M -32 -10 Q -46 -4 -38 14 Q -30 8 -26 -2 Z" fill="#F05138" stroke="#dc2626" stroke-width="1.5"/>
  <path d="M 32 -10 Q 46 -4 38 14 Q 30 8 26 -2 Z" fill="#F05138" stroke="#dc2626" stroke-width="1.5"/>
  <line x1="-10" y1="30" x2="-10" y2="42" stroke="#f97316" stroke-width="3" stroke-linecap="round"/>
  <line x1="10" y1="30" x2="10" y2="42" stroke="#f97316" stroke-width="3" stroke-linecap="round"/>
</g>
<g id="elder">
  <ellipse cx="0" cy="60" rx="40" ry="6" fill="#1e293b" opacity="0.12"/>
  <path d="M -30 58 L 30 58 L 24 2 L -24 2 Z" fill="#7C3AED" stroke="#5b21b6" stroke-width="2"/>
  <rect x="-20" y="20" width="40" height="6" rx="3" fill="#a78bfa"/>
  <circle cx="0" cy="-22" r="30" fill="#f8fafc" stroke="#cbd5e1" stroke-width="2.5"/>
  <path d="M -22 -26 Q -24 -40 -10 -44 L 10 -44 Q 24 -40 22 -26" fill="#4c1d95"/>
  <circle cx="-10" cy="-25" r="7" fill="none" stroke="#64748b" stroke-width="1.8"/>
  <circle cx="10" cy="-25" r="7" fill="none" stroke="#64748b" stroke-width="1.8"/>
  <line x1="-3" y1="-25" x2="3" y2="-25" stroke="#64748b" stroke-width="1.8"/>
  <circle cx="-10" cy="-25" r="3" fill="#007AFF"/>
  <circle cx="10" cy="-25" r="3" fill="#007AFF"/>
  <path d="M -10 -8 Q 0 -4 10 -8" stroke="#94a3b8" stroke-width="2" fill="none" stroke-linecap="round"/>
  <path d="M -14 -6 Q -12 8 -6 14 L 6 14 Q 12 8 14 -6" fill="#f1f5f9" stroke="#cbd5e1" stroke-width="1.5"/>
  <line x1="0" y1="-54" x2="0" y2="-72" stroke="#64748b" stroke-width="3"/>
  <circle cx="0" cy="-76" r="6" fill="#F05138" stroke="#1e293b" stroke-width="1.5"/>
  <path d="M 0 -82 L -4 -76 L 4 -76 Z" fill="#22c55e"/>
</g>
```

使用：格内 `<use href="#swifty" transform="translate(x,y) scale(0.5)"/>`；知识条固定 `<use href="#elder" transform="translate(150,744) scale(0.62)"/>`。

## 5. 内容与写作规范

- 三格叙事：格1 呈现一个真实痛点（崩溃/事故现场），格2 拆机制（图示 + 关键 API），格3 收束到可落地清单或代码。
- 知识条固定三行「Runtime 长老划重点」，每行 ≤ 34 个汉字，动词开头、结论式表述。
- 讲解 MD（45–100 行）小节固定：登场角色 / 剧情梗概 / 分格解读 / Runtime 长老划重点 / 自测一题 / 去实验室 / 下一话预告；必须相对链接 `../<章号>-*.md` 与本话 `./EP.xx-*.svg`。
- 可见文本禁止裸 `<` `>` `&`（用 `&lt;` `&gt;` `&amp;`），箭头用 `→`，不用 emoji。
- 徽章文本 `EP.xx` 必须出现；代码字号 11.5px，每行 ≤ 62 字符。
