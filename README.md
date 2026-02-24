### ruleset

用于 Clash / Mihomo / HomeProxy 的规则与配置模板。

推荐模板：

- `rule/config/Clash/通用版.yaml`（已去除个人订阅，使用 `机场1/订阅1` 占位）
- `rule/config/Clash/openclash_compatible.yaml`（OpenClash 兼容版，Meta/Premium 均可）
- `rule/config/Clash/config.yaml`（单订阅基础模板）

说明：

- 本仓库模板不包含真实订阅链接。
- 使用前请将 `订阅1/订阅2/订阅3` 替换为你自己的订阅地址。

兼容说明：

- Clash Verge / FlClash：可直接导入 `通用版.yaml`
- luci-app-fchomo / luci-app-momo / luci-app-nikki：可直接导入 `通用版.yaml`
- luci-app-openclash：请使用 Meta(Mihomo) 内核；Clash Premium 内核不支持 mrs 规则格式
- 若需 OpenClash Premium 兼容，请使用 `openclash_compatible.yaml`
