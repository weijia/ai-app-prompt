# CSV 拖拽出图工具

> 拖进 CSV → 自动识别字段 → 一键成图并可导出图片 · 难度：★★☆ · 约 3 分钟

## 适用场景

手上有个 Excel/CSV，不想打开专业软件，拖进来就能看趋势和分布。

## 占位符说明

| 占位符 | 含义 | 示例 |
| --- | --- | --- |
| `{{数据主题}}` | 数据大概是什么 | 「门店每日客流」 |
| `{{常用图表}}` | 优先展示哪些图 | 「折线、柱状、饼图」 |

## 提示词（中文版）

````text
请生成一个「CSV 拖拽出图工具」单文件网页应用。

【背景】
我的数据主题：{{数据主题}}
常用图表：{{常用图表}}

【功能要求】
1. 拖拽区：支持把 .csv 文件拖进来（dragover/drop 事件需 preventDefault 阻止浏览器直接打开文件），也支持点击选择文件；同时提供粘贴文本与内置示例数据。
2. CSV 解析器必须自己实现并正确处理：
   - 引号包裹字段 "xxx, yyy"
   - 字段内换行
   - 分隔符自动识别（逗号 / 制表符 / 分号，取第一行出现次数最多的）
   - 中文 GBK 编码文件给出「若出现乱码请另存为 UTF-8」的提示
3. 列类型自动推断：数值列 / 日期列 / 文本列，推断结果允许手动改（下拉切换）。
4. 图表配置面板：X 轴选择、Y 轴多选、聚合方式（求和 / 平均 / 计数 / 最大值）、图表类型切换。
5. 图表渲染后立即提供「下载 PNG」按钮（canvas 或 SVG 转 PNG）。
6. 数据统计摘要：对数值列显示最大值、最小值、平均值、中位数、空值数量。
7. 数据预览表格：显示前 50 行，异常值（非数字出现在数值列）用黄色背景标出。

【依赖策略（严格遵守）】
只可使用 ECharts，地址固定为 https://cdn.staticfile.org/echarts/5.5.0/echarts.min.js（版本号写死）。
必须实现 onerror 降级：若加载失败，则改用纯 CSS 实现的横向条形图（用 div 宽度百分比表示数值大小）展示同样的聚合结果，并显示黄色提示条，绝不白屏。
下载 PNG 在降级模式下改为「导出 SVG/CSV」，不要报错。
禁止使用 jsdelivr / unpkg / cdnjs 及其余第三方库。

【交付格式】
只输出一个完整 index.html，HTML/CSS/JS 全部内联，不用 npm 与构建工具。
回复结构：一句话说明 → 唯一的 ```html 完整代码块（禁止省略号）→ 3 行使用说明。

【工程要求】
1. 界面全中文，移动端优先响应式（含 viewport meta）；375px 宽下拖拽区高度不小于 160px，配置面板折叠成抽屉。
2. 最近一次数据用 localStorage 持久化，key 前缀 "csvdash-"（注意大小限制，超过 2MB 时跳过持久化并提示）。
3. 处理：空文件、只有表头、全部列都无法识别等情况的友好提示。
4. 视觉：现代简洁、虚线拖拽框 hover 高亮、拖拽悬停时整体模糊变暗、prefers-color-scheme 暗色模式。
5. 代码行数控制在 650 行以内，CSV 解析器加中文注释说明状态机逻辑。
````

## Prompt (English Version)

````text
Generate a single-file "Drag & Drop CSV Chart" web app.

[CONTEXT]
My data is about: {{TOPIC}}
Preferred charts: {{CHART_TYPES}}

[FEATURES]
1. Drop zone accepting .csv files (preventDefault on dragover/drop so the browser doesn't open the file), plus click-to-select, paste-text input and built-in sample data.
2. Write your own CSV parser that correctly handles:
   - quoted fields "xxx, yyy"
   - newlines inside quoted fields
   - delimiter auto-detection (comma / tab / semicolon — pick the one most frequent in the first line)
   - if Chinese text looks garbled, show "Save the file as UTF-8 and retry"
3. Infer column types (numeric / date / text) and let the user override each with a dropdown.
4. Chart config panel: X axis picker, Y axis multi-select, aggregation (sum / avg / count / max), chart type switch.
5. "Download PNG" button right after rendering (canvas or SVG -> PNG).
6. Summary stats per numeric column: max, min, mean, median, null count.
7. Data preview table for the first 50 rows; highlight anomalous values (non-numbers in a numeric column) with a yellow background.

[DEPENDENCIES — STRICT]
ECharts only, pinned: https://cdn.staticfile.org/echarts/5.5.0/echarts.min.js
Implement an onerror fallback: on failure, render the same aggregation with pure CSS horizontal bars (div width as a percentage) and show a yellow banner. Never a blank page.
In fallback mode, "Download PNG" becomes "Export SVG/CSV" instead of throwing.
No jsdelivr / unpkg / cdnjs, no other libraries.

[DELIVERY]
One complete index.html, all inline. No npm, no build tools.
Reply: one-line summary -> exactly ONE ```html full block (no ellipsis) -> 3 usage lines.

[QUALITY]
1. English UI, mobile-first responsive with viewport meta; at 375px the drop zone is at least 160px tall and the config panel collapses into a drawer.
2. Persist the last dataset in localStorage with key prefix "csvdash-" (skip persistence above 2MB and notify the user).
3. Friendly handling of empty files, header-only files, and columns that can't be typed.
4. Modern clean visuals: dashed drop zone that highlights on hover, dimmed backdrop while dragging, prefers-color-scheme dark mode.
5. Keep under 650 lines; comment the CSV parser state machine.
````

## 生成后自检清单

- [ ] 拖文件时浏览器没有直接打开它，而是被页面接收
- [ ] 含逗号引号的 CSV 行没有被错误切分
- [ ] 断网时降级为 CSS 条形图而不是白屏
- [ ] 下载 PNG 得到的图片不是全黑
- [ ] 超过 2MB 的文件不会卡死页面

## 常用追问

1. 「支持同时导入多个 CSV 做对比」
2. 「加散点图和相关系数计算」
3. 「把清洗后的数据导出为 JSON」
