# 我马上来 — IP 定位科幻闪动页

打开页面自动获取访问者的城市位置，显示 `bro,你在XX省XX市对吧，我马上来`，伴随约 1 秒的科幻故障（glitch）入场闪动特效。纯静态页面，部署于 GitHub Pages。

---

## 效果预览

- **入场动画（约 1 秒）**：CRT 扫描光带扫过全屏 → 文字强 glitch 故障抖动（RGB 色偏）→ 字符乱码逐位解码为最终文字 → 扫描线渐隐稳定
- **稳定后**：霓虹青绿色文字居中显示，四角装饰框，低透明度 CRT 扫描线常驻
- **"bro," 微交互**：颜色与正文略有差异，点击跳转 B 站主页；文字稳定 3 秒后触发一次 0.28 秒的崩坏闪烁
- **定位**：强制走 IPv4，五层降级链，国内城市优先用高德（准），国外/失败降级 ipwho.is

---

## 功能特性

- [x] IP 定位自动填入城市名
- [x] 强制 IPv4（避免 IPv6 地址库漂移到错误城市）
- [x] 五层 API 降级兜底
- [x] 科幻 glitch 入场动画（约 1 秒）
- [x] CRT 扫描线 + 扫描光带
- [x] "bro," 可点击跳转 + 3 秒后崩坏闪烁
- [x] 移动端响应式
- [x] 纯静态，零依赖，零构建

---

## 技术栈

| 层 | 技术 |
|---|---|
| 页面 | 纯 HTML + CSS + 原生 JS（无框架、无构建工具） |
| 定位（IPv4 获取） | [myip.ipip.net](https://myip.ipip.net/)（主力）/ [ipv4.icanhazip.com](https://ipv4.icanhazip.com/)（备用） |
| 定位（地理查询） | [高德地图 V3 IP 定位](https://lbs.amap.com/api/webservice/guide/api/ipconfig/)（主力）/ [ipwho.is](https://ipwho.is/)（降级） |
| 部署 | GitHub Pages |

---

## 快速开始

### 本地预览

直接用浏览器打开 `index.html` 即可，无需任何构建或服务器。

> 注意：部分浏览器对 `file://` 协议下的 `fetch` 有限制，如果定位不工作，用本地 HTTP 服务器打开：
> ```bash
> python3 -m http.server 8080
> # 然后访问 http://localhost:8080
> ```

### 部署到 GitHub Pages

1. 将 `index.html` 推送到 GitHub 仓库的 `main` 分支根目录
2. 仓库 Settings → Pages → Source 选 `Deploy from a branch` → Branch 选 `main` / `/root`
3. 等待 1-2 分钟，访问 `https://<用户名>.github.io/<仓库名>/`

---

## 代码结构

整个项目只有一个文件 `index.html`，内联 CSS 和 JS。

### HTML 结构

```
<body>
├── .scanlines        # CRT 扫描线覆盖层（固定全屏）
├── .scan-beam        # 入场扫描光带
├── .corner.tl/tr/bl/br  # 四角装饰框
├── .container
│   └── #message      # 主文字（居中，动态拼接）
└── #status           # 底部状态文字（BOOTING / CONNECTING / CONNECTED）
```

### JS 函数清单

| 函数 | 行号（约） | 职责 |
|---|---|---|
| `randChar()` | 234 | 从乱码字符集随机取一个字符 |
| `setText(txt)` | 238 | 设置文字内容 + 同步 `data-text` 属性（供 glitch 伪元素用） |
| `decodeTo(target, duration)` | 244 | 字符乱码逐位解码动画，`duration` 毫秒内完成 |
| `formatLocation(province, city)` | 274 | 地名格式化：直辖市直接用 province，省/自治区拼接为 `省+市+市` |
| `enhanceBro()` | 289 | 稳定后把 "bro," 包裹为可点击 span + 3 秒后崩坏闪烁 |
| `getIPv4()` | 315 | 获取用户纯 IPv4 地址（myip.ipip.net 主力 → icanhazip 备用） |
| `requestAmap(ip)` | 353 | 请求高德 V3 IP 定位，传 `ip` 参数则查指定 IP，不传则查请求方 IP |
| `requestIpwho(ip)` | 375 | 请求 ipwho.is 定位，传 `ip` 查指定 IP，不传查请求方 IP |
| `locate()` | 398 | **主定位函数**，五层降级链，返回 `{province, city}` 或 `null` |
| `boot()` | 412 | **入场主流程**，控制动画时序 + 调用定位 + 拼接文字 |
| `wait(ms)` | — | Promise 版 setTimeout |

### 关键变量

| 变量 | 说明 |
|---|---|
| `AMAP_KEY` | 高德地图 Web 服务 API Key（需自行申请） |
| `GLITCH_CHARS` | 乱码解码动画用的字符集 |
| `FALLBACK_TEXT` | 定位全部失败时显示的兜底文案 |

---

## 自定义指南

### 1. 修改显示文字

文字模板在 `boot()` 函数中拼接：

```javascript
// 找到这行（约在 boot 函数内）
target = locText ? ('bro,你在' + locText + '对吧，我马上来') : FALLBACK_TEXT;
```

修改引号内的文字即可。例如改成 `你好，你在' + locText + '，对吗？`。

兜底文案修改 `FALLBACK_TEXT` 变量（第 232 行附近）。

### 2. 修改定位 API

#### 更换高德 Key

修改 `AMAP_KEY` 变量（第 224 行）：

```javascript
var AMAP_KEY = '你的新key';
```

申请地址：https://lbs.amap.com/ → 控制台 → 创建应用 → 添加 Key（服务平台选"Web 服务"）

#### 更换 IPv4 获取服务

在 `getIPv4()` 函数的 `services` 数组中增删：

```javascript
var services = [
  {
    url: 'https://你的服务地址/',
    parse: function (text) {
      // 从返回文本中提取 IPv4，返回字符串或 null
      var m = text.match(/(\d{1,3}\.){3}\d{1,3}/);
      return m ? m[0] : null;
    }
  },
  // ... 更多服务，按优先级排列
];
```

要求：服务必须支持 CORS（`Access-Control-Allow-Origin: *`），且最好是仅 IPv4 域名（无 AAAA 记录）。

#### 更换地理定位服务

修改 `requestAmap()` 或 `requestIpwho()` 函数，或在 `locate()` 的降级链中增删层级。

`locate()` 当前降级顺序：
```
1. getIPv4() → requestAmap(ipv4)     （主力：IPv4 + 高德）
2. getIPv4() → requestIpwho(ipv4)     （高德失败时）
3. requestAmap(null)                   （无 IPv4 时，直连高德）
4. requestIpwho(null)                  （高德失败时，直连 ipwho）
5. return null                         （全部失败，走兜底文案）
```

### 3. 修改入场特效

#### 调整动画总时长

在 `boot()` 函数中调整各个 `await wait(xxx)` 的数值：

```javascript
await wait(180);   // 初始闪烁时长
// ...
decodeTo(target, 350);  // 解码动画时长
await wait(380);         // 等待解码完成
```

#### 调整 glitch 抖动强度

在 CSS 中修改 `@keyframes glitch-shake`、`glitch-red`、`glitch-blue` 的 `translate` 数值。数值越大抖动越剧烈。

#### 调整乱码字符集

修改 `GLITCH_CHARS` 变量，增删字符。

#### 调整扫描线

CSS `.scanlines` 的 `repeating-linear-gradient` 控制扫描线密度，`.scanlines.fade` 的 `opacity` 控制稳定后的透明度。

### 4. 修改 "bro," 交互

#### 修改跳转链接

在 `enhanceBro()` 函数中找到：

```javascript
window.open('https://space.bilibili.com/627411857?...', '_blank');
```

替换为目标 URL。

#### 修改崩坏闪烁触发时间

在 `enhanceBro()` 中找到 `setTimeout(function () { ... }, 3000);`，修改 `3000`（毫秒）。

#### 修改崩坏闪烁时长/效果

CSS 中 `@keyframes bro-break` 控制崩坏动画，`.bro-link.bro-glitch` 的 `animation` 属性控制时长（当前 `0.28s`）。

#### 修改 bro 颜色

CSS `.bro-link` 的 `color` 属性（当前 `#88ffcc`，比主文字 `#00ffaa` 略亮）。

### 5. 修改颜色主题

| 元素 | CSS 变量/属性 | 当前值 |
|---|---|---|
| 背景 | `body::before` 的 `radial-gradient` | `#0a1f14` → `#050505` |
| 主文字色 | `.message` 的 `color` | `#00ffaa`（霓虹青绿） |
| 主文字辉光 | `.message` 的 `text-shadow` | `rgba(0,255,170,...)` |
| glitch 红色 | `.message.glitch::before` 的 `color` | `#ff2d55` |
| glitch 蓝色 | `.message.glitch::after` 的 `color` | `#00c8ff` |
| 扫描线/装饰框 | `.scanlines` / `.corner` | `rgba(0,255,170,...)` |
| bro 颜色 | `.bro-link` 的 `color` | `#88ffcc` |
| 底部状态 | `.status` 的 `color` | `rgba(0,255,170,0.35)` |

全局替换 `#00ffaa` 和 `rgba(0,255,170` 可以快速切换主色调。

### 6. 修改字号/响应式

`.message` 的 `font-size: clamp(1.3rem, 4.5vw, 3.2rem);` 控制响应式字号：
- 最小值 `1.3rem`（小屏）
- 首选值 `4.5vw`（随视口宽度缩放）
- 最大值 `3.2rem`（大屏上限）

---

## API 说明

### 高德地图 V3 IP 定位

- **文档**：https://lbs.amap.com/api/webservice/guide/api/ipconfig/
- **地址**：`https://restapi.amap.com/v3/ip?key=KEY[&ip=IP]`
- **特点**：国内 IP 准确率高，仅支持 IPv4，不支持国外 IP
- **免费额度**：5000 次/天
- **传 `ip` 参数**：查询指定 IP；不传则查询请求方 IP
- **返回**：`{ status, province, city, adcode, rectangle }`

### myip.ipip.net

- **地址**：`https://myip.ipip.net/`
- **特点**：国内服务（~0.34s），仅 IPv4 域名（无 AAAA 记录），支持 CORS
- **返回**：纯文本 `当前 IP：x.x.x.x  来自于：中国 省 市  运营商`
- **免费**：无需注册，无需 key

### ipv4.icanhazip.com

- **地址**：`https://ipv4.icanhazip.com/`
- **特点**：Cloudflare 运营（~0.9s），仅 IPv4 域名，支持 CORS
- **返回**：纯文本 IP 地址
- **免费**：无需注册

### ipwho.is

- **文档**：https://ipwho.is/
- **地址**：`https://ipwho.is/[IP]?lang=zh-CN`
- **特点**：支持指定 IP 查询，支持国内外 IP，支持中文
- **免费**：无需注册，无明确速率限制

---

## 常见问题

### Q: 为什么定位不准？

A: IP 定位只能到城市级，且存在固有误差：
- **IPv6 地址库**：国内 IPv6 部署时间短，很多地市的 IPv6 段被登记到省会或邻市。本项目已强制走 IPv4 规避此问题。
- **移动网络**：4G/5G 的出口 IP 经常在省内甚至跨省漂移，这是运营商的问题，任何 IP 定位都解决不了。
- **中小城市**：国外 IP 库（ipwho.is）对三四线城市经常归到省会。本项目主力用高德（国内库），准确率更高。

### Q: 手机和电脑显示不同城市？

A: 检查手机是否用的移动数据而非 WiFi。移动数据的出口 IP 和宽带完全不同，显示不同城市是正常现象。连同一个 WiFi 应该显示一致。

### Q: 高德 Key 会泄露吗？

A: 会。纯静态页面的 Key 写在前端 JS 里，任何人查看源码都能看到。降低风险的方法：
- 在高德控制台设置 Key 的使用限制（Web 服务 Key 仅支持 IP 白名单，但 GitHub Pages 出口 IP 不固定，无法设置）
- 定期监控调用量，异常时删除重建 Key
- 如需彻底隐藏，需用 Cloudflare Workers 等后端代理

### Q: 入场动画可以关掉吗？

A: 可以。在 `boot()` 函数中，把 `decodeTo(target, 350)` 改为 `setText(target)`，并移除前面的 `flicker`/`glitch` 类操作，即可直接显示文字无动画。

### Q: 如何添加更多降级 API？

A: 在 `getIPv4()` 的 `services` 数组中添加 IPv4 获取服务，或在 `locate()` 中添加地理定位服务的降级层级。参考"自定义指南 → 修改定位 API"。

---

## 许可证

MIT
