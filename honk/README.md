# Honk 配置说明

Honk 是 dae 的 Rust + eBPF 重写，配置语法和 dae 同源（文件都叫 `config.dae`），但有三处结构性差异。

| 文件 | 说明 |
|------|------|
| `config.dae` | 策略组 + DNS + 分流规则 |
| `system.dae` | `global` / `experimental`，由 `config.dae` 用 `include` 引入 |

## 和 dae 的三个差异

**1. global 可以拆出去（也可以不拆）**

honk 内核两种都吃：`global` / `experimental` 直接写在 `config.dae` 里能跑，拆到 `system.dae` 用 `include` 引入也能跑。本仓库选择拆分，是因为配 `luci-app-honk` 时，LuCI 需要改「监听端口 / Clash API」这类内核参数，而它只会去改 `system.dae`，不会去动用户的内容文件：

```dae
include {
    system.dae
}
```

> 网上很多教程（例如 qichiyuhub/rule）是**单文件**写法，`global` 和 `experimental` 全写在 config.dae 里。那种配置也能直接用：在 LuCI「配置 → 导入配置」上传即可，导入后是**手动模式**，配置由文件说了算，LuCI 的端口 / Clash API 设置会变只读。

**2. DNS upstream 定义时就绑出口**

```dae
# honk
upstream {
    default: 'https://223.5.5.5/dns-query?tls_server_name=dns.alidns.com' -> direct
    remote:  'https://8.8.8.8/dns-query?tls_server_name=dns.google'        -> Proxy
}

# dae
upstream {
    remote: 'https://8.8.8.8/dns-query'
    local:  'https://223.5.5.5/dns-query'
}
```

honk 的 `default` / `remote` 是在定义 DNS 服务器那一刻就指定了走 direct 还是走某个组，dae 是只定义服务器、出口由别处决定。

**3. 多了 `experimental.native_api`**

Doona 面板靠它连核心，dae 没有这一段。

## 两种运行模式

配 LuCI 时，同一份 honk 有两套管理模式，切换入口在「配置」页顶部：

| | 托管模式（默认） | 手动模式 |
|---|---|---|
| 谁决定配置 | `system.dae`（内核参数）+ `config.dae`（分流） | 只有 `config.dae`，整份文件说了算 |
| 怎么进入 | 初始化后默认就是 | 上传/粘贴整份配置，点「保存并应用」 |
| 添加订阅 / 粘贴节点 | 可用（写进 config.dae 的订阅段） | **置灰不可用** |
| 端口 / Clash API 开关 / 网卡 | LuCI 设置页可改 | 只读，由 config.dae 决定 |
| 恢复出厂 / 修复 | 有效 | 「恢复托管模式」会换回上次的托管快照 |

判断标准很简单：**要在 LuCI 里改端口、开关 Clash API、用订阅工具 → 托管模式；只想拿别人写好的整份配置直接跑 → 手动模式。**

## 部署到 252

```bash
# 1. 本机改好配置，推到本仓库
# 2. 传上路由器
ssh daed252 'mkdir -p /etc/honk'
tar czf - config.dae system.dae | ssh daed252 'tar xzf - -C /etc/honk'

# 3. 校验配置（honk-core 的 config validate，会输出 JSON 诊断）
ssh daed252 '/usr/bin/honk-core config validate --config /etc/honk/config.dae'

# 4. 重启并看日志
ssh daed252 '/etc/init.d/honk restart; sleep 2; logread | tail -30'
```

`scp` 在这台设备上可能因为缺 `sftp-server` 失败，用上面的 `tar | ssh` 走 stdin。

## 依赖

| 需要 | 说明 |
|------|------|
| `/usr/share/v2ray/geosite.dat` | `v2ray-geosite` 包 |
| `/usr/share/v2ray/geoip.dat` | `v2ray-geoip` 包 |
| `/sys/kernel/btf/vmlinux` | eBPF CO-RE 必需，缺了 honk 起不来 |
| `kmod-sched-core` / `kmod-sched-bpf` / `kmod-veth` / `kmod-nft-queue` | eBPF 调度 |

配置里用到的 geosite 代码和 [dae 那份](../dae/config.dae) 完全一致，来源是 Loyalsoldier 的 v2ray-rules-dat。

## 常见问题

**起不来，日志没有任何输出**
先查 `/sys/kernel/btf/vmlinux` 存不存在。Honk 用 CO-RE eBPF，内核没开 BTF 就是静默失败。252 是 kernel 6.12，自带 BTF，正常。

**Doona 面板打不开**
`experimental.native_api.listen` 是 `127.0.0.1:9527`，只能本机访问。LuCI 那边有反代；如果直接拿浏览器开，需要 SSH 端口转发。

**改了配置没生效**
honk 的配置在 `/etc/honk/`，改本仓库的文件不等于改路由器上的文件。要走上面「部署到 252」的步骤。
