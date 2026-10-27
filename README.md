# ruleset

Clash / Mihomo / dae / sing-box / HomeProxy 的规则与配置模板集合。

面向个人自用，不保证通用。用之前先看清楚每个文件是给哪个客户端用的。

- Clash 系用 `rule/config/mihomo/`
- dae 用 `rule/config/dae/`
- sing-box 用 `rule/config/singbox/`
- 规则名单统一放 `rule/*.list`，配置里用 raw URL 引用
- 图标放 `icon/`

---

## 一、规则名单（`rule/*.list`）

这些是本仓库自己维护的名单，配置里通过 raw URL 引用。改完推到 `main` 就生效。

| 文件 | 格式 | 内容 | 主要用途 |
|------|------|------|----------|
| `rule/Direct.list` | classical | 520 条：阿里 / 字节 / 腾讯 / B 站 / 网易等国内大厂域名 | 直连补充。防「域名走代理」导致国内站变慢或风控 |
| `rule/ProxyLite.list` | classical | 26 条：Plex、国外 DNS 服务器 IP | 代理补充。Plex 远程访问、DoH 服务器必须走代理 |
| `rule/AI.list` | classical | Perplexity / Grok / Copilot / HuggingFace / Poe / Cursor 等 | AI 兜底组。OpenAI / Claude / Gemini 已由上游 mrs 覆盖，这份管「其余 AI 站」 |
| `rule/ApplePush.list` | classical | APNs 推送域名 + Apple 推送 IP 段 | 独立推送组。国内用 iPhone 建议 APNs 走代理，否则推送延迟或丢 |
| `rule/FakeIPFilter-CN.list` | domain text | 局域网、国内 NTP、运营商一键登录、国内音乐/游戏/Apple 更新 CDN | fake-ip 白名单（国内段）。命中则返回真实 IP |
| `rule/FakeIPFilter-NotCN.list` | domain text | 国外 NTP、STUN、Switch/PS/Xbox 联机、Netflix、Disney+、Plex | fake-ip 白名单（国外段）。同上 |

**为什么要 fake-ip 白名单**：fake-ip 模式会给每个域名返回假的 `198.18.x.x`。时间同步、STUN 打洞、游戏机联机、局域网设备这几类如果拿到假 IP，表现是「网页能开，但时间不更新 / 联机失败 / 找不到 NAS」。所以必须放行真实解析。

直接引用地址（替换 `main` 为其他分支即切换）：

```
https://raw.githubusercontent.com/kenzok78/ruleset/main/rule/Direct.list
https://raw.githubusercontent.com/kenzok78/ruleset/main/rule/ProxyLite.list
https://raw.githubusercontent.com/kenzok78/ruleset/main/rule/AI.list
https://raw.githubusercontent.com/kenzok78/ruleset/main/rule/ApplePush.list
https://raw.githubusercontent.com/kenzok78/ruleset/main/rule/FakeIPFilter-CN.list
https://raw.githubusercontent.com/kenzok78/ruleset/main/rule/FakeIPFilter-NotCN.list
```

> `raw.githubusercontent.com` 被墙时，把前缀换成 `https://cdn.jsdelivr.net/gh/kenzok78/ruleset@main/`。

---

## 二、Mihomo / Clash（`rule/config/mihomo/`）

| 文件 | 适合谁 | 说明 |
|------|--------|------|
| `config.yaml` | **推荐，大多数人用这个** | 多订阅 + 完整分流 + 图标 + fake-ip 白名单。走 fake-ip 但不强制 DNS 全走代理，追求「能上、能分流」 |
| `config-fakeip.yaml` | 在意 DNS 泄漏 | 分流规则和 `config.yaml` 一样，只换 DNS：国内域名走国内 DNS，国外域名走国外 DNS 并从代理出去，输出前做 geoip 校验防污染。DNS 一旦不通会表现为「全部打不开」，排查成本高 |
| `config-basic.yaml` | 单订阅、想快速跑通 | 只有一个机场占位，规则也少 |
| `config-full.yaml` | 想要全端口 + 系统级配置 | 开了 redir / tproxy / socks 等全端口，带 geox-url 和自动更新 geo |
| `config-compat.yaml` | Clash Premium 内核 | 全部改用 text/yaml 规则，不用 mrs 格式。Meta 内核请用 `config.yaml` |
| `kenzo.yaml` | 自用 | 我的个人模板，订阅已换成占位符 |
| `kenzo.ini` / `kenzo_mini.ini` | OpenClash / Stash 的规则集定义 | `.ini` 不是 Clash 配置，是客户端里的「规则集订阅」格式 |
| `fx.yaml` | 带端口转发的旧版模板 | 保留兼容，不建议新用 |

### 客户端适配

| 客户端 | 用哪个 |
|--------|--------|
| Clash Verge Rev | `config.yaml` |
| FlClash | `config.yaml` |
| luci-app-fchomo / luci-app-momo / luci-app-nikki | `config.yaml` |
| luci-app-openclash（Meta/Mihomo 内核） | `config.yaml` |
| luci-app-openclash（Premium 内核） | `config-compat.yaml` |

### 关于图标

`proxy-groups` 里的 `icon:` 字段需要 mihomo v1.18.0+，Clash Verge Rev / zashboard / FlClash / metacubexd 能显示。老客户端会直接忽略这个字段，不影响使用。

图标是自托管的，源文件在 `icon/`：

- 品牌 logo：来自 [homarr-labs/dashboard-icons](https://github.com/homarr-labs/dashboard-icons)（Apache-2.0）
- 国家旗帜：来自 [lipis/flag-icons](https://github.com/lipis/flag-icons)（MIT）
- 通用图标（rocket / direct / home / globe / fish / auto / all / bell / film）：本仓库自绘

想让某个组有图标，加一行 `icon: "https://raw.githubusercontent.com/kenzok78/ruleset/main/icon/<名字>.svg"`。

---

## 三、dae（`rule/config/dae/`）

| 文件 | 说明 |
|------|------|
| `config.dae` | 完整版。AI 分成 OpenAI / Claude / Gemini / AI 四组，平台各自分组，国内外 DNS 分流 |
| `config.min.dae` | 精简版。只有一个 Proxy 组，先跑通再说 |

### 依赖的 geo 数据

dae 的 `geosite:` / `geoip:` 规则必须要有这两个文件：

```
/usr/share/v2ray/geosite.dat
/usr/share/v2ray/geoip.dat
```

OpenWrt 上装 `v2ray-geosite` + `v2ray-geoip` 即可，`luci-app-daede` 的「更新 geo」按钮会下载 [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) 的版本。

配置里用到的 geosite 代码（已核对存在）：`cn`、`geolocation-!cn`、`private`、`openai`、`anthropic`、`google-gemini`、`category-ai-!cn`、`youtube`、`google`、`github`、`onedrive`、`microsoft`、`microsoft@cn`、`apple`、`apple@cn`、`telegram`、`tiktok`、`netflix`、`paypal`、`binance`、`okx`、`category-cryptocurrency`、`steam`、`steam@cn`。

### 三个必须改的地方

1. `subscription` 里的 `sub` —— dae **只支持 v2ray / URL 格式订阅**，mihomo 格式要先用 sub-store 转
2. `global` 里的 `lan_interface` —— x86 软路由一般是 `br-lan`，用 `ip a` 确认
3. `group` 里的地区正则 —— 改成你机场节点名里真实出现的关键字

### dae 和 mihomo 的两个硬差异

- 组名只能是英文和数字，**不能用中文和 emoji**
- 没有 `fake-ip-filter` 这个概念，靠 `dns.routing` 里按域名分国内 / 国外 DNS 来避免污染

---

## 四、sing-box / HomeProxy（`rule/config/singbox/`）

官方内核 1.12 / 1.13 / 1.14 各版本的客户端配置，以及 HomeProxy 用的版本。

1.13 之后官方内核支持 `proxy-providers` 之前，客户端配置要靠 sub-store 远程脚本填充，脚本地址见各版本目录里的 `*-substore-scripts.md`。

---

## 五、其他客户端

| 目录 | 说明 |
|------|------|
| `rule/config/shadowrocket/` | Shadowrocket 配置（防 DNS 泄漏） |
| `rule/config/openclash/` | OpenClash 的 UCI 配置，**不是** Clash 配置，放 `/etc/config/openclash` |
| `icon/` | 策略组图标，见上文「关于图标」 |
