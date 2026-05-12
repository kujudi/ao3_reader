# Project Instructions — Archive Style Reader

## 项目概述

**目标**：让阅读体验尽可能接近 AO3（Archive of Our Own）自己的网页浏览体验。这是一个仿 AO3 风格的本地阅读器，以单个 HTML 文件分发，用户将 `.txt`、`.md`、`.epub` 文件拖入浏览器即可阅读，零安装、零后端、完全本地运行。

**项目名称**：Archive Style Reader  
**主文件**：`ao3-reader.html`  
**技术栈**：纯前端，单文件 HTML + CSS + JavaScript，无构建步骤，无后端  
**外部依赖**：JSZip 3.10.1（仅 epub 功能需要，从 cdnjs.cloudflare.com 加载，离线时 epub 不可用，txt/md 不受影响）

---

## 文件结构

```
ao3_reader/
├── ao3-reader.html              ← 主文件（全部功能在此）
├── Instruction.md               ← 本文档
│
│   ── 参考素材（AO3 官方 CSS，只读，不参与构建）──
├── 1_site_screen_.css
├── 4_site_midsize.handheld_.css
├── 5_site_narrow.handheld_.css
├── 6_site_speech_.css
└── 7_site_print_.css
```

`ao3-reader.html` 内部三段式结构：

```
ao3-reader.html
├── <style>     CSS（CSS 变量 + 完整布局/组件样式）
├── <body>      HTML（上传页 #upload-section + 阅读页 #work-section）
└── <script>    所有业务逻辑，原生 JS，无框架
```

### JS 函数结构

```
文件处理
├── handleFile(file)          入口，按扩展名分发到 txt 或 epub 处理器
├── handleText(file)          处理 .txt / .md，含编码自动检测
├── handleEpub(file)          处理 .epub，异步，依赖 JSZip
├── detectEncoding(bytes)     二进制检测 UTF-8 / GBK / UTF-16
└── setFileInfo(name, str)    更新文件信息栏 UI

epub 解析
├── handleEpub()              解压 zip → 读 container.xml → 读 OPF → 按 spine 提取章节
└── parseEpubHtml(html)       剥离 HTML 标签，提取章节标题和正文纯文本

渲染
├── renderWork()              读取表单元数据，整合章节，渲染作品页
├── displayChapter(idx)       渲染指定章节内容，更新导航状态
├── textToHtml(text)          纯文本/Markdown 转 HTML（支持段落、标题、分隔线、加粗斜体）
└── escapeHtml(str)           XSS 防护

视图切换
├── showWork()                切换到阅读视图
└── showUpload()              切换回上传视图

工具函数
├── escapeRegex(str)          正则转义
├── ratingLabel(r)            评级代码转显示文字
├── stripTags(html)           剥离 HTML 标签
└── formatSize(bytes)         文件大小格式化
```

---

## 设计系统

### 目标

以 AO3 官方页面为基准，在视觉和交互上尽量还原，包括：导航栏颜色与阴影、按钮渐变样式、元数据框 dt/dd 布局、tag 虚线下划线、Summary/Notes 左侧竖线等细节。参考素材是项目根目录下的 AO3 官方 CSS 文件。

### 主题色（CSS 变量）

```css
--red: #990000          /* 主色，来自 AO3 */
--red-dark: #700000     /* 悬停深色 */
--red-nav: #800000      /* 导航栏 */
--bg-page: #ffffff      /* 页面背景 */
--bg-inner: #f5f5f0     /* 内容区浅米色背景 */
--border: #dddddd
--border-light: #e0e0e0
--text: #2a2a2a
--text-muted: #666666
--text-hint: #999999
--link: #2a5a8e
--link-red: #990000
```

### 字体

```css
--font-serif: Georgia, 'Times New Roman', serif          /* 正文 */
--font-sans: Arial, 'PingFang SC', 'Microsoft YaHei', sans-serif  /* UI */
```

正文（`.chapter-body`）使用 AO3 原版字体栈：`'Lucida Grande', 'Lucida Sans Unicode', Verdana, Helvetica`，行高 1.5，这与 AO3 实际渲染一致。

### 布局

最大宽度 72em，居中。阅读页切换到 `.reading-mode` 时去除左右 padding，元数据框与正文各自独立居中，与 AO3 结构对齐。

---

## 功能说明

### 支持的格式

| 格式 | 编码检测 | 章节处理 |
|------|---------|---------|
| `.txt` | 自动（UTF-8 BOM / UTF-16 / GBK 启发式） | 按分隔符拆分，或整篇一章 |
| `.md`  | 同上 | 同上，支持 Markdown 语法渲染 |
| `.epub` | N/A（epub 内部是 UTF-8） | 自动读取 OPF spine 顺序 |

### epub 解析流程

1. JSZip 解压文件
2. 读 `META-INF/container.xml` → 找到 OPF 文件路径
3. 解析 OPF manifest（属性顺序无关，支持自闭合标签 `/>` 和普通 `>`）
4. 按 spine 顺序取各章 HTML 文件
5. `parseEpubHtml()` 剥离标签，提取纯文本
6. 过滤掉正文少于 20 字的页面（封面、版权页等）
7. 自动填入书名、作者字段

### 元数据表单

用户填写后渲染到 AO3 风格的元数据框（dl/dt/dd 浮动两列）：Rating、Archive Warning、Categories、Fandom、Relationships、Characters、Additional Tags、Language、Published 日期、Stats（字数/章节数/Kudos 等统计行）。

### 阅读界面（AO3 风格还原）

- 顶部白色 header + 红色主导航栏（带 box-shadow 立体感）
- 元数据框（浮动两列，tag 用虚线下划线，hover 变红底白字）
- 顶部 / 底部按钮栏（灰色渐变按钮）：`Entire Work`、`Chapter Index`（下拉菜单，当前章高亮）、`Comments`、`Share`、`Download`、`← Previous Chapter`、`Next Chapter →`、`Kudos`
- Summary 模块（左侧红色竖线）、Notes 模块（左侧灰色竖线）
- 章节正文（AO3 字体栈，居中单列，最大 72em）
- 章节标题居中，下方细分隔线
- 评论区（含留言表单 + 字符倒计数，提交后显示本地提示）
- 面包屑导航，浏览器 `<title>` 随作品同步更新

### 交互功能

- 键盘左/右/上/下箭头切换章节
- Kudos 按钮（本地状态，点击后变色锁定）
- 滚动到顶部 / 滚动到评论区
- 视图切换：上传页 ↔ 阅读页（点击 logo 或"+ 上传新文章"）
- Markdown 渲染：`#`/`##`/`###` 标题、段落、`---` 分隔线、`**加粗**`、`*斜体*`
- 响应式：`max-width: 620px` 断点，双列表单变单列

---

## 已知限制与注意事项

- **epub 需要联网**加载 JSZip，离线时 epub 功能不可用（txt/md 完全离线）
- epub 章节按 spine 顺序拆分，每个 HTML 文件是一个条目；Calibre 处理过的 epub 会把同一章拆成多个 `part000X_split_00Y.html`，章节数会偏多
- txt 的章节分隔符是正则匹配，填入的字符串被转义后用作正则前缀，复杂分隔符（如纯数字）可能需要测试
- 没有持久化存储，刷新页面或关闭标签后阅读进度丢失
- 点赞按钮和评论是装饰性的，数据不保存
- epub 内嵌图片当前被忽略，只提取文字

---

## 后续可扩展的方向

1. **阅读进度持久化**：用 `localStorage` 记录当前章节和滚动位置，下次打开自动跳转
2. **书库页面**：允许用户在同一标签里管理多本书（需要 IndexedDB 存储文件）
3. **GitHub Pages 部署**：文件可以直接放到 GitHub Pages，无需服务器
4. **epub 图片支持**：提取 epub 内的插图并在正文中展示
5. **txt 自动识别章节**：启发式自动识别常见中文章节标题格式，无需用户填写分隔符
