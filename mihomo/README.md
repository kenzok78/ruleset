# Mihomo / Clash 配置说明

| 文件 | 适合谁 | 说明 |
|------|--------|------|
| `config.yaml` | **推荐** | 多订阅 + 完整分流 + 图标 + fake-ip 白名单 |
| `config-fakeip.yaml` | 在意 DNS 泄漏 | 只换 DNS 部分：国内走国内 DNS，国外走国外 DNS + 指定代理出口 |
| `config-basic.yaml` | 单订阅、快速跑通 | 机场只有一个，规则也少 |
| `config-full.yaml` | 想要全端口 + 系统级配置 | redir / tproxy / socks 全开，带 geox-url 自动更新 |
| `config-compat.yaml` | Clash Premium 内核 | 全部 text/yaml 规则，不用 mrs |
| `kenzo.yaml` | 自用 | 个人模板，订阅已占位化 |
| `openclash/` | OpenClash 专用 | `openclash.uci` 是 UCI 配置，`kenzo.ini` / `kenzo_mini.ini` 是 OpenClash 的自定义规则集订阅格式，都不是 Clash 配置 |
| `fx.yaml` | 覆写模板（Overwrite） | **不含订阅和节点**，只有 global / dns / tun / proxy-groups / rules。配合 Clash Verge 的「覆写 / Overwrite」或 OpenClash 的「覆写设置」叠加到自己的配置上，直连目标统一写 `DIRECT` 而不是 `🌐 直连`，因为覆写时不确定主配置有没有定义后者 |

## 客户端对应

- Clash Verge Rev / FlClash → `config.yaml`
- luci-app-fchomo / momo / nikki → `config.yaml`
- luci-app-openclash + Meta 内核 → `config.yaml`
- luci-app-openclash + Premium 内核 → `config-compat.yaml`

## 使用步骤

1. 选一个文件
2. 把 `订阅1` / `订阅2` / `订阅3` 替换成自己的机场订阅链接
3. 用不到的机场，把 `proxy-providers` 里对应那段和 `provider-anchor` 里的名字一起删掉
4. 导入客户端

## 常见问题

**提示规则格式不支持**
Premium 内核加载了 `config.yaml`。两个办法：换 Meta(Mihomo) 内核，或改用 `config-compat.yaml`。

**没有节点**
订阅链接没替换，或者链接在当前网络下拿不到。先确认 `proxy-providers` 的 `url` 能直接打开。

**图标不显示**
mihomo 需要 v1.18.0+，客户端需要支持 `icon` 字段（Clash Verge Rev / zashboard / FlClash / metacubexd）。老客户端会忽略，不影响分流。

**国内站变慢或需要验证码**
把域名加进 `../rule/Direct.list`，或者检查有没有开「拦 QUIC」那条规则。

**NTP 对时失败 / 游戏联机失败 / 找不到 NAS**
fake-ip 白名单没覆盖到。把域名加进 `../rule/FakeIPFilter-CN.list` 或 `../rule/FakeIPFilter-NotCN.list`。

**日志出现 `global-client-fingerprint is removed`**
mihomo v1.19 起移除了全局指纹配置。新内核不用管；旧内核需要全局伪装时自己加回 `global-client-fingerprint: chrome`。
