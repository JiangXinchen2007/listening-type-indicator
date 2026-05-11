# EI 情绪智能测试网站 · 开发日志

> 上次更新：2026-05-11  
> 对话轮次：已完成 Round 1–6 优化

---

## 项目基本信息

| 项目 | 内容 |
|------|------|
| 课程 | 大学情绪智能（EI）课程第二阶段小组作业 |
| 形式 | 纯静态单文件 HTML（无框架、无后端、无外部依赖） |
| 部署 | 可直接双击浏览器打开（`file://`），也可部署到 GitHub Pages |
| 主文件（只读原版） | `/Users/jxc/Desktop/EI/第二阶段/index.html`（797行，暗色 glassmorphism 风格） |
| 当前开发文件 | `/Users/jxc/Desktop/EI/第二阶段/index_v2.html`（插画风改版，Round 5 最新） |

---

## 网站功能概述

**「你是哪种倾听动物？」** — 基于 Zenger & Folkman（2016）六级倾听量表的趣味心理测试

### 流程（5页切换）
```
首页(Hero) → 介绍页(Intro) → 测试页(Quiz×8题) → 加载页(Loading) → 结果页(Result)
```

### 6种测试结果（动物角色）

| 动物角色 | 英文称号 | 科学类型 | 颜色 | 稀有度 |
|---------|---------|---------|------|------|
| 🐋 深海聆听鲸 | THE DEEP LISTENER | 海绵型 | `#60A5FA` | ★★★★★ 极其稀有 |
| 🦉 月夜共情枭 | THE EMPATHY SAGE | 镜子型 | `#C084FC` | ★★★★★ 极其稀有 |
| 🐬 彩虹活力豚 | THE VIBE AMPLIFIER | 蹦床型 | `#2DD4BF` | ★★★☆☆ 活跃常见 |
| 🐯 闪电行动虎 | THE FIXER | 解决者型 | `#FB923C` | ★★★☆☆ 行动常见 |
| 🐻 暖阳陪伴熊 | THE WARM COMPANION | 陪伴型 | `#F472B6` | ★★★★☆ 珍稀暖心 |
| 🦋 探索好奇蝶 | THE CURIOUS QUESTIONER | 探问型 | `#FBBF24` | ★★★★★ 极其稀有 |

---

## 技术架构

### CSS 设计系统（Round 5 当前）
```css
:root {
  --bg: #FDF8F0;           /* 暖奶油色主背景 */
  --bg2: #F0FFF4;          /* 浅绿辅助背景（介绍页） */
  --card: #FFFFFF;
  --tx: #18110A;           /* 深棕黑主文字 */
  --dim: #4A3828;          /* 次要文字 */
  --mu: #9B8878;           /* 辅助文字 */
  --pri: #18A449;          /* 主色：Duolingo 绿 */
  --pri-d: #0D5C2A;        /* 绿色阴影（按钮底部） */
  --pri-l: #DCFCE7;        /* 浅绿高亮 */
  /* 6种动物色（主色/深色/浅色） */
  --whale: #60A5FA; --whale-d: #1D4ED8; --whale-l: #EFF6FF;
  --owl:   #C084FC; --owl-d:   #7C3AED; --owl-l:   #F5F3FF;
  --dolphin:#2DD4BF;--dolphin-d:#0F766E;--dolphin-l:#F0FDFA;
  --tiger: #FB923C; --tiger-d: #C2410C; --tiger-l: #FFF7ED;
  --bear:  #F472B6; --bear-d:  #BE185D; --bear-l:  #FFF0F7;
  --butterfly:#FBBF24;--butterfly-d:#B45309;--butterfly-l:#FFFBEB;
}
```

### 页面切换系统
```css
.page { position: fixed; inset: 0; opacity: 0; pointer-events: none; transition: opacity .4s ease }
.page.active { opacity: 1; pointer-events: all }
```
通过 JS `showPage(id)` 切换 `.active` class 实现无路由多页。

### Duolingo 风格按钮
```css
.btn-p { background: var(--pri); box-shadow: 0 5px 0 var(--pri-d) }
.btn-p:active { transform: translateY(4px); box-shadow: 0 1px 0 var(--pri-d) }
```

### SVG 动物系统（Round 5：扁平几何风格）
- **风格**：无重描边，大色块叠压，简洁几何形状，保留专属配件
- **实现**：隐藏 `<svg><defs>` 块中定义 6 个 `<g id="svg-xxx">`，用 `<use href="#svg-xxx"/>` 引用
- **viewBox**：统一 `0 0 200 200`（蝴蝶触角已修复，安全在 viewBox 内）
- **尺寸使用**：首页卡 56px、介绍页卡 38px、结果页大展示 144px、分数条 16px、Canvas 分享图 340px

### 3D 翻转卡片系统（Round 5 新增）
```css
.acard { perspective: 800px; min-height: 158px; cursor: pointer }
.acard-inner { transform-style: preserve-3d; transition: transform .55s cubic-bezier(.34,1.1,.64,1) }
.acard.flipped .acard-inner { transform: rotateY(180deg) }
.acard-front, .acard-back { position: absolute; inset: 0; backface-visibility: hidden }
.acard-back { transform: rotateY(180deg) }
/* 介绍页 .tp 同理，用 .tp-inner/.tp-front/.tp-back */
```
点击 → toggle `.flipped` → 背面展示该类型标语 + 倾听级别。

### 首页入场动效（Round 5 新增）
```css
@keyframes title-zoom { from{transform:scale(2.4);opacity:0} 30%{opacity:1} to{transform:scale(1);opacity:1} }
#p-hero.active .hero-h1 { animation: title-zoom 1.1s cubic-bezier(.22,1,.36,1) both }
/* 副标题 .75s、按钮 .85s、动物格 .95s 依次淡入 */
```

### 介绍页入场动效（Round 5 新增）
```css
#p-intro.active .lv     { animation: fu .4s var(--dl,.1s) both }
#p-intro.active .lv-fill{ width: var(--w); transition: width .9s ease-out calc(var(--dl,0s) + .3s) }
#p-intro.active .tp     { animation: fu .38s var(--dl,.12s) both }
```
HTML 中每项通过 `style="--dl:Xs;--w:X%"` 传入 stagger 延迟和进度条目标宽度。

### 结果页 heroBg 渐变系统（Round 5 更新）
```js
// buildResult() 中
r-hero.style.background = `linear-gradient(to bottom, ${t.heroBg} 0%, ${t.heroBg} 65%, #FDF8F0 100%)`
```
上65%为纯深色，下渐变至奶油色，与内容区无缝衔接。

### Canvas 分享图（Round 5 更新）
- `saveImg()` 改为 `async`，用 `drawAnimalOnCanvas()` Promise 将 SVG 渲染到 Canvas
- 渐变末端保持深色（`heroBg + 'DD'`）
- 底部文字可读性：`.20→.55`、`.35→.80`、`.5→.78`

### 计分系统（8题×3选项，完全均衡）

| 题号 | A | B | C |
|------|---|---|---|
| Q1 | 海绵 | 镜子 | 解决者 |
| Q2 | 陪伴 | 蹦床 | 探问 |
| Q3 | 海绵 | 陪伴 | 蹦床 |
| Q4 | 解决者 | 镜子 | 探问 |
| Q5 | 海绵 | 陪伴 | 解决者 |
| Q6 | 蹦床 | 镜子 | 探问 |
| Q7 | 海绵 | 镜子 | 蹦床 |
| Q8 | 解决者 | 陪伴 | 探问 |

每种类型各出现 **4次**，完全均衡。**并列优先级**：`镜子 > 探问 > 海绵 > 陪伴 > 蹦床 > 解决者`

### 核心 JS 函数
| 函数 | 作用 |
|------|------|
| `showPage(id)` | 切换页面 |
| `startQuiz()` | 重置分数，开始测试 |
| `renderQ(n)` | 渲染第n题 |
| `pick(tp, el, n)` | 记录答案，380ms后跳下一题 |
| `quizBack()` | 返回上一题并撤销计分 |
| `doLoad()` | 进入加载页，2.6s后计算结果 |
| `buildResult(k)` | 填充结果页所有内容 |
| `animalSVG(type, size)` | 生成 SVG use 元素字符串 |
| `drawAnimalOnCanvas(ctx, svgId, x, y, w, h)` | 异步将 SVG 绘制到 Canvas |
| `confetti(col)` | 生成彩带动画（70粒） |
| `saveImg()` | async，Canvas 生成 1080×1350 PNG 并下载 |
| `retake()` | 重置所有状态，回首页 |

---

## 已完成的优化轮次

### Round 1（初版）
- 基础 5 页流程，4 种倾听类型，分享功能

### Round 2（大改版）
- 4 种动物角色命名，暗色 glassmorphism 风格，Canvas 图片下载

### Round 3（6种动物完整版）
- 新增陪伴熊 + 好奇蝶，8题均衡覆盖6种类型

### Round 4（插画风 UI 大改版）
- 浅色 Duolingo 风格，绿色主色，SVG 动物（重描边+大眼睛），heroBg 深色背景系统

### Round 6（布局精修 + 渐变可读性 + 入场动效升级）✅ 当前最新
- **新增** 首页 Splash 入场：大字横版标题全屏显示，向下滑动（或点击）触发退出动效，标题缩小上移淡出，主页内容浮现
- **新增** 实体「返回」按钮：白色背景 + 阴影 + 按下效果，替换原文字链接
- **升级** 六阶段模块：全部六级使用绿色，级别1→6 递进渐变（rgba .015 → .20），深浅各不相同
- **修复** 结果页渐变可读性：r-hero 改为纯色深背景（100% 不透明），新增独立 `.r-grad`（200px 纯渐变过渡区，无文字），彻底解决渐变区文字不可读问题
- **优化** 介绍页标题排版：改为「倾听，/ 也有六个段位」两行
- **优化** 介绍页右栏间距：动物卡片 gap 9→14px，卡片高度 128→142px，「开始测试」按钮 margin-top 24px
- **删除** 介绍页「8道情境题 · 匿名完成 · 约3分钟」小字

### Round 5（动效 + 交互 + 质量全面提升）
- **删除** hero-meta 小字
- **修复** 蝴蝶 SVG 触角剪裁（触角球从 cy=2 移至安全位置）
- **新增** 首页入场动效（标题 2.4x 缩放进场缩回，其他元素延迟淡入）
- **新增** 动物卡片 3D 翻转（首页 acard + 介绍页 tp，背面展示标语+级别）
- **重绘** 全部 6 只 SVG（扁平几何风格，无重描边，大色块）
- **新增** 介绍页入场动效（六级 stagger 展开，进度条从0增长，动物卡错落淡入）
- **修复** 介绍页六级统一模块样式（全部6级有卡片样式，高亮级加绿色）
- **修复** 介绍页右栏下移 padding-top:48px
- **修复** 结果页颜色衔接（hero 渐变过渡至奶油色）
- **升级** Canvas 分享图（SVG 卡通形象替换 emoji，深色渐变，文字可读性提升）

---

## 给下一个对话的交接说明

1. **直接阅读本文件**即可了解当前状态，无需重新探索代码
2. **只修改 `index_v2.html`**，`index.html` 是只读原版，永远不动
3. 测试方法：直接双击 `index_v2.html` 用浏览器打开（`file://`），不需要本地服务器
4. **当前阶段**：Round 5 已完成，可继续收集截图反馈进行 Round 6
5. 代码风格：CSS 单行压缩，JS 保留可读性，HTML 保持缩进
6. 翻转卡片：`.acard` / `.tp` 点击触发 `this.classList.toggle('flipped')`
7. 动效触发：依赖 `.page.active` CSS 选择器，`showPage()` 添加 `.active` 即触发
8. 如需查阅题目数据、T 对象、JS 函数完整定义，参见 `index_v2.html` 的 `<script>` 段

## 文件结构

```
/Users/jxc/Desktop/EI/第二阶段/
├── index.html      ← 原版（只读，暗色 glassmorphism，797行）
├── index_v2.html   ← 当前开发版（插画风，Round 5 最新）
└── DEV_LOG.md      ← 本文件
```
