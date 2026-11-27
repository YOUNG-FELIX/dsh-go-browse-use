# dsh-go-browse-use

在 dsh-go 引擎里跑浏览器桥接服务，并注册 `browser` 工具：读页面、抓链接、截图、点按钮。Firefox 与支持扩展的 Chromium 系（Chrome / Edge / Vivaldi 等）都可用。

> 本仓库托管**浏览器扩展安装包与安装说明**；插件本体通过 npm 分发（`dsh-go-browse-use`），也可以直接在 dsh 市场里搜索「浏览器桥接」。

## 版本对应

| 组件 | 版本 | 分发方式 |
| --- | --- | --- |
| 插件（引擎侧） | 1.8.0 | npm [`dsh-go-browse-use`](https://www.npmjs.com/package/dsh-go-browse-use)、dsh 市场 |
| Firefox 扩展 | 1.12.0 | 本仓库 Releases 的 `.xpi`（需签名后安装） |
| Chromium 扩展 | 1.12.0 | 本仓库 Releases 的 `.zip`（解压加载）或 `.crx` |

插件与扩展分开版本：插件负责起服务、注册 `browser` 工具和面板；扩展负责在浏览器里执行命令。

## 下载

到 [Releases](https://github.com/YOUNG-FELIX/dsh-go-browse-use/releases/latest) 下载：

- `dsh-go-browse-use-1.12.0.xpi` —— Firefox
- `dsh-go-browse-use-1.12.0-chromium.zip` —— Chrome / Edge / Vivaldi 解压加载
- `dsh-go-browse-use-1.12.0.crx` —— Edge Canary / Dev 等允许从文件安装的 Chromium

## 安装

### 一、安装插件

在 dsh 市场里搜索「浏览器桥接」安装，或使用命令行：

```bash
dsh plugin --profile web add dsh-go-browse-use
```

离线安装：直接下载本仓库的 [`plugin/dsh-go-browse-use-1.8.0.tgz`](plugin/dsh-go-browse-use-1.8.0.tgz)，再本地安装：

```bash
dsh plugin --profile web add ./dsh-go-browse-use-1.8.0.tgz
```

安装后重启引擎（插件 `lib/` 改动必须重启才生效）。

### 二、安装浏览器扩展

扩展要和插件连的是同一台机器的 `127.0.0.1:8790`，浏览器装在本机即可。

#### Firefox

Firefox 正式版只允许安装**已签名**的扩展，目前还未上架 AMO，需要二选一：

1. **自己签名（推荐，装完长期可用）**
   - 打开 https://addons.mozilla.org/developers/addon/submit/
   - 选择「On your own site（自己分发）」，上传 `dsh-go-browse-use-1.12.0.xpi`
   - 签名完成后下载签好的 `.xpi`，在 Firefox 里打开 `about:addons` → 齿轮 → 「从文件安装附加组件」，选择签好的 xpi
2. **临时加载（只用于调试，重启浏览器失效）**
   - 打开 `about:debugging#/runtime/this-firefox`
   - 「临时载入附加组件」→ 选择 `dsh-go-browse-use-1.12.0.xpi`

> 上架 AMO 审核通过后，可以直接在 addons.mozilla.org 搜索安装，届时本节会更新。

#### Chromium 系（Chrome / Edge / Vivaldi）

- **解压加载（通用）**：把 `dsh-go-browse-use-1.12.0-chromium.zip` 解压到固定目录，打开 `chrome://extensions`（Edge 是 `edge://extensions`），开启「开发者模式」，点「加载已解压的扩展程序」，选择解压出的目录。
- **Edge Canary / Dev**：可直接把 `dsh-go-browse-use-1.12.0.crx` 拖进扩展页面安装（浏览器需允许从文件安装 CRX）。
- Chrome 正式版不支持直接拖入 CRX，请用上面的解压加载方式。

> 上架 Chrome Web Store / Edge Add-ons 审核通过后，可以直接在商店搜索安装，届时本节会更新。

### 三、配对

1. 引擎里打开插件面板（插件入口 → 浏览器桥接），点「打开桥接」；或让助手执行 `browser({ action: 'on' })`
2. 读取面板里的 6 位配对码（10 分钟有效、一次性），或让助手执行 `browser({ action: 'paircode' })`
3. 点浏览器里的扩展图标 → 状态页 → 输入配对码
4. 配对成功后自动重连，之后不用再配

每个安装实例用自己的配对码换取一对独立令牌，令牌存在浏览器本地；吊销其中一个不影响其他实例。

## 端口与开关

- 端口固定 **8790**（不用 8765，避免和已有 MCP 服务冲突）
- 桥接**默认关闭**，不常驻；关掉会立刻释放端口
- 开关状态在 `$DSH_HOME/dsh-go/browser-bridge/enabled.json`，重启引擎后保持

## 常用动作

```js
browser({ action: 'tabs' })
browser({ action: 'text', tab: '3' })
browser({ action: 'snapshot' })              // 可交互元素清单（带 aria-ref，省上下文）
browser({ action: 'click', selector: 'aria-ref=e3' })
browser({ action: 'goto', url: 'https://example.com' })
browser({ action: 'shot' })
```

`snapshot` 只返回能点的元素和 `ref`，后续 `click` / `fill` / `wait` 直接用 `aria-ref=eN` 定位，不用整页 HTML 或截图占上下文。

## 安全模型

- **扩展层**：cookie 只出门元数据，真值留在浏览器侧
- **桥接层**：命令级授权，敏感命令进不了队列；cookie 落盘 `0600`；正文与诊断日志转发时脱敏
- **插件层**：走 App 原生审批弹窗，用户不点「允许」工具不会执行
- `cookies.get` 只返回 `{name, length, fp, domain, path, secure, httpOnly, sameSite, session, expirationDate}`，不含 `value`

## License

MIT，见 [LICENSE](LICENSE)。
