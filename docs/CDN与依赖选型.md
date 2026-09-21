# CDN 与依赖选型说明

本库所有提示词默认执行下面的依赖策略，目的是**保证国内网络环境下打开就能用**。

## 1. 总原则：零依赖优先

原生 API 能做的（DOM 操作、localStorage、Canvas、定时、动画、表单、加减乘除图表），**一律不引第三方库**。
原因：

1. 豆包 / 元宝生成的代码里，第三方库的 API 写错概率远高于原生代码；
2. 少一个外链就少一个加载失败点；
3. 单文件离线可保存、可直接发微信给别人。

## 2. 确实需要库时的 CDN 优先级

| 优先级 | 源 | 示例 | 说明 |
| --- | --- | --- | --- |
| 1 | Staticfile CDN | `https://cdn.staticfile.org/echarts/5.5.0/echarts.min.js` | 国内节点稳定，资源较全 |
| 2 | 360 baomitu | `https://lib.baomitu.com/echarts/5.5.0/echarts.min.js` | 备选源，速度好 |
| 3 | BootCDN | `https://cdn.bootcdn.net/ajax/libs/echarts/5.5.0/echarts.min.js` | 老牌国内源，注意版本号路径 |
| 4 | 字节跳动公共库 | `https://lf*-cdn-tos.bytecdntp.com/...` | 变动频繁，需在 [字节 Web CDN](https://www.bytecdntp.com/) 查准确路径 |
| 5 | npmmirror | `https://registry.npmmirror.com/<pkg>/<ver>/files/<file>` | 兜底直取 npm 包文件 |

**明确禁用**（国内访问可能超时或被阻断，导致页面直接白屏）：

- `cdn.jsdelivr.net` / `fastly.jsdelivr.net`
- `unpkg.com`
- `cdnjs.cloudflare.com`

## 3. 必须写降级逻辑

凡是用了 CDN 的提示词，都要求模型生成如下兜底结构：

```html
<script src="https://cdn.staticfile.org/echarts/5.5.0/echarts.min.js" onerror="this.dataset.failed=1"></script>
<script>
  window.addEventListener('load', function () {
    var s = document.querySelector('script[src*="echarts"]');
    if (!window.echarts || (s && s.dataset.failed)) {
      // 降级：用原生 <table> 渲染同样的数据，并给出提示
      renderFallbackTable(DATA);
      showBanner('图表库加载失败，已切换为表格视图（不影响数据查看）');
    } else {
      renderChart(DATA);
    }
  });
</script>
```

做不到降级的场景（如 OCR、复杂地理可视化），应改为**不依赖 CDN 的替代方案**。

## 4. 常见第三方能力替代方案

| 想要的能力 | 推荐做法 |
| --- | --- |
| 图表 | ECharts（CDN + 表格降级）；简单柱状/进度条用纯 CSS 自己画 |
| 二维码生成 | 纯 JS 手写 QR 编码器（约 80 行，零依赖），或用 Canvas 版本；避免依赖 qrcodejs 外网源 |
| Markdown 渲染 | 写一个 200 行内的极简正则解析器；不要用 marked CDN |
| 拼音/中文处理 | 用 `Intl` / `localeCompare` 原生能力 |
| 日期处理 | 原生 `Date` + 简单格式化函数，不用 dayjs |
| 数据导出 CSV | `Blob` + `URL.createObjectURL`，零依赖 |
| 图片压缩/裁剪 | 原生 `Canvas`，零依赖 |
| 动画 | CSS transition / Web Animations API，不用 anime.js |
| 状态管理 | 原生对象 + 手写 render 函数，不用 Vue/React |

## 5. 版本锁定

所有 CDN 地址都要求**写死具体版本号**（如 `echarts/5.5.0/`），禁止使用 `/latest/`、`@latest` 之类浮动路径，以免 CDN 更新导致 API 变化而崩掉。

## 6. 网络请求提醒

若生成结果里出现 `fetch('https://...')` 调用第三方 API（如汇率、天气），务必注意：

- 该域名在国内是否可访问；
- 是否需要跨域（CORS）许可；
- 不要在任何请求中携带用户隐私数据。

本库涉及网络的提示词（如汇率换算）均要求「接口失败时降级为手动输入汇率」。
