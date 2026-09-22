# JSON 格式化工具

> 校验 + 美化 + 折叠树 + 路径复制 · 难度：★★★ · 约 3 分钟

## 占位符说明

| 占位符 | 含义 | 示例 |
| --- | --- | --- |
| `{{缩进}}` | 缩进几空格 | 「2」 |
| `{{常用操作}}` | 最需要什么 | 「压缩、排序键、转义/去转义、转表格」 |

## 提示词（中文版）

````text
请生成一个「JSON 格式化工具」单文件网页应用，全部在浏览器本地处理，不上传数据。

【配置】
默认缩进：{{缩进}} 空格
我常用的操作：{{常用操作}}

【功能要求】
1. 左右双栏：左侧输入（textarea，等宽字体、显示行号），右侧结果展示；窄屏改为上下分区 + 标签切换。
2. 核心能力：
   - 美化（缩进可选 2/4/Tab）
   - 压缩（去空格换行）
   - 校验：解析失败时用 JSON.parse 的错误信息定位出错位置，并在编辑器中标红对应行、给出中文解释（如「第 3 行第 12 列附近缺少逗号或右括号」）
   - 排序键（对象 key 按字典序递归排序）
   - 转义 / 去转义
   - JSON ↔ URL 参数串互转（注意 URL 编码处理）
3. 树形视图：把 JSON 渲染为可折叠树（递归 DOM 生成），对象/数组可点击展开收起，数组显示元素个数，超过 100 个子节点时默认折叠并提示；不同类型用不同颜色区分（字符串绿、数字蓝、布尔紫、null 灰）。
4. 路径复制：鼠标悬停或点击某个节点显示其 JSON Path（如 `data.items[2].name`），一键复制该路径或该路径对应的值。
5. 搜索：输入关键词在所有 key 与 string 值中查找，命中项高亮并自动展开其父节点，显示命中数量与上一个/下一个跳转。
6. 统计：节点总数、最大嵌套深度、字符数与字节数（用 TextEncoder 计算 UTF-8 字节数）。
7. 导出：下载 .json 文件、复制到剪贴板、从剪贴板粘贴导入。

【安全要求】
不要使用 eval 或 new Function 解析 JSON，必须使用 JSON.parse。渲染所有字符串值时必须做 HTML 转义，防止内容里包含 HTML 被执行。

【依赖策略】
纯原生 JavaScript，零第三方库，零 CDN，禁止引入 CodeMirror / jsoneditor 等编辑器库。

【交付格式】
只输出一个完整 index.html，HTML/CSS/JS 全部内联，不用 npm 与构建工具。
回复结构：一句话说明 → 唯一的 ```html 完整代码块（禁止省略号）→ 3 行使用说明。

【工程要求】
1. 界面全中文，移动端优先响应式（含 viewport meta），375px 宽可用；输入框字号不小于 14px 防 iOS 自动缩放。
2. 最近一次内容用 localStorage 持久化，key 前缀 "jsontool-"（内容超过 200KB 时不持久化并提示）。
3. 处理：超大 JSON（> 1MB）时的处理提示与分阶段渲染（避免页面假死）、空输入提示、非法 JSON 的详细报错。
4. 视觉：类似编辑器的暗色/浅色双主题（默认跟随系统）、行号与代码区对齐、语法配色清晰、prefers-color-scheme 暗色模式。
5. 页面显眼位置注明「所有处理均在你的浏览器本地完成，数据不会上传」。
6. 代码总量建议控制在 700 行左右；功能完整性优先于行数——不得为压缩行数而省略功能或用省略号代替代码，超出时只精简注释与冗余写法。树渲染与错误定位部分加中文注释。
````

## Prompt (English Version)

````text
Generate a single-file "JSON Formatter" web app. Everything runs locally in the browser; nothing is uploaded.

[CONFIG]
Default indent: {{INDENT}} spaces
Operations I use most: {{OPS}}

[FEATURES]
1. Two panes: input textarea on the left (monospace, line numbers) and results on the right; on narrow screens stack vertically with tabs.
2. Core operations:
   - pretty print with 2 / 4 / Tab indent options
   - minify (strip whitespace and newlines)
   - validation: on parse failure, use the JSON.parse error to locate the problem, highlight the offending line in red, and explain it in plain English (e.g. "around line 3, column 12: missing comma or closing brace")
   - sort object keys recursively (lexicographic)
   - escape / unescape
   - convert JSON <-> URL query string (handle URL encoding correctly)
3. Tree view: render the JSON as a collapsible tree built recursively in the DOM; objects/arrays expand and collapse, arrays show their length, nodes with more than 100 children start collapsed with a notice; color-code types (strings green, numbers blue, booleans purple, null gray).
4. Path copy: hovering or clicking a node reveals its JSON Path (e.g. `data.items[2].name`) with one-click copy of the path or its value.
5. Search: find a keyword across all keys and string values, highlight matches, auto-expand their parents, and offer hit counts with prev/next navigation.
6. Stats: node count, max nesting depth, character count and UTF-8 byte size via TextEncoder.
7. Export: download .json, copy to clipboard, paste-import from clipboard.

[SECURITY]
Never parse with eval or new Function — always JSON.parse. HTML-escape every rendered string value so embedded HTML can't execute.

[DEPENDENCIES]
Vanilla JS only. Zero libraries, zero CDN. Do not load CodeMirror, jsoneditor or any editor library.

[DELIVERY]
One complete index.html, all inline. No npm, no build tools.
Reply: one-line summary -> exactly ONE ```html full block (no ellipsis) -> 3 usage lines.

[QUALITY]
1. English UI, mobile-first responsive with viewport meta, usable at 375px; input font at least 14px to prevent iOS auto-zoom.
2. Persist the last content in localStorage with key prefix "jsontool-" (skip persistence above 200KB and say so).
3. Handle oversized JSON (> 1MB) with a notice and staged rendering to avoid freezing, plus empty-input guidance and detailed invalid-JSON errors.
4. Editor-like light/dark themes (follow the system by default), aligned line numbers, clear syntax colors, prefers-color-scheme dark mode.
5. Prominently state: "Everything is processed locally in your browser; nothing is uploaded."
6. Aim for about 700 lines; completeness outranks length — never drop features or use ellipsis to save lines; if it runs over, trim comments and redundancy only. Comment the tree rendering and error localization.
````

## 生成后自检清单

- [ ] 粘贴一段错误 JSON 能指出第几行出错
- [ ] 1000 行以上的大 JSON 不会卡死浏览器
- [ ] 树视图能折叠展开且 JSON Path 正确
- [ ] 内容里的 `<script>` 字符串不会被渲染执行
- [ ] 刷新页面后上次内容还在

## 常用追问

1. 「加对比功能：左右两份 JSON 做 diff」
2. 「加 JSON Schema 校验（简易必填字段检查）」
3. 「支持导出为 Excel CSV（针对对象数组）」
