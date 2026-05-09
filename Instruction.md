# Project Instructions — Archive Style Reader

## 项目概述

这是一个仿 AO3（Archive of Our Own）风格的本地阅读器，以单个 HTML 文件的形式分发，用户无需安装任何软件，在浏览器中直接打开即可使用。项目面向有阅读需求的个人用户，也可以分享给他人使用。

**项目名称**：Archive Style Reader  
**主文件**：`ao3-reader.html`  
**技术栈**：纯前端，单文件 HTML + CSS + JavaScript，无构建步骤，无后端  
**外部依赖**：JSZip 3.10.1（仅 epub 功能需要，从 cdnjs.cloudflare.com 加载，离线时 epub 不可用，txt/md 不受影响）

---

## 文件结构

项目是单文件架构，所有代码在 `ao3-reader.html` 内分三个部分：

```
ao3-reader.html
├── <style>          CSS 样式，使用 CSS 变量统一管理主题色和字体
├── HTML 结构        两个主视图：上传页（#upload-section）和阅读页（#work-section）
└── <script>         所有逻辑，无框架，原生 JS
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

### 主题色（CSS 变量）

```css
--red: #990000          /* 主色，来自 AO3 */
--red-dark: #770000     /* 悬停深色 */
--red-nav: #800000      /* 导航栏 */
--bg-page: #f5f5f0      /* 页面背景，米白色 */
--bg-white: #ffffff     /* 卡片背景 */
```

### 字体

```css
--font-serif: Georgia, 'Times New Roman', 'Noto Serif SC', serif   /* 正文 */
--font-sans: Arial, 'PingFang SC', 'Microsoft YaHei', sans-serif   /* UI */
```

正文使用衬线字体，行高 1.85，这是 AO3 舒适阅读体验的核心。

### 布局

最大宽度 900px，居中。内容区实际文字宽度约 700px，符合中文长文阅读的最佳行宽。

---

## 功能说明

### 支持的格式

| 格式 | 编码检测 | 章节处理 |
|------|---------|---------|
| `.txt` | 自动（UTF-8 / GBK / UTF-16） | 按分隔符手动拆分，或整篇一章 |
| `.md` | 同上 | 同上，支持 Markdown 语法渲染 |
| `.epub` | N/A（epub 内部是 UTF-8） | 自动读取 OPF spine 顺序 |

### epub 解析流程

1. JSZip 解压文件
2. 读 `META-INF/container.xml` → 找到 OPF 文件路径
3. 解析 OPF manifest（属性顺序无关，支持自闭合标签 `/>` 和普通 `>`）
4. 按 spine 顺序取各章 HTML 文件
5. `parseEpubHtml()` 剥离标签，提取纯文本
6. 过滤掉正文少于 20 字的页面（封面、版权页等）
7. 自动填入书名、作者字段

### 阅读器功能

- 上下章导航（按钮 + 下拉菜单 + 键盘方向键）
- 字号调节（13–22px 滑块）
- 行距切换（紧凑 / 舒适 / 宽松）
- 字体切换（宋体风格 / 黑体风格 / 楷体风格）
- 面包屑导航，浏览器标题同步更新
- 点赞按钮（纯本地状态，无持久化）

---

## 已知限制与注意事项

- **epub 需要联网**加载 JSZip，离线时 epub 功能不可用（txt/md 完全离线）
- epub 章节是按 spine 顺序拆分的，每个 HTML 文件是一个条目；部分由 calibre 处理过的 epub 会把同一章拆成多个 `part000X_split_00Y.html`，这种情况下章节数会偏多
- txt 的章节分隔符是正则匹配，填入的字符串会被转义后用作正则前缀，复杂分隔符（如纯数字）可能需要测试
- 没有持久化存储，刷新页面或关闭标签后阅读进度丢失
- 点赞按钮是装饰性的，数据不保存

---

## 后续可扩展的方向

如果需要继续开发，以下是优先级较高的方向：

1. **阅读进度持久化**：用 `localStorage` 记录当前章节和滚动位置，下次打开自动跳转
2. **夜间模式**：CSS 变量已经集中管理，只需增加一套暗色变量和切换逻辑
3. **书库页面**：允许用户在同一个标签里管理多本书（需要 IndexedDB 存储文件）
4. **GitHub Pages 部署**：文件可以直接放到 GitHub Pages，无需服务器，用户访问链接即可使用
5. **epub 图片支持**：当前版本只提取文字，epub 内的插图被忽略
6. **txt 自动识别章节**：当前需要用户填写分隔符，可以尝试启发式自动识别常见中文章节标题格式