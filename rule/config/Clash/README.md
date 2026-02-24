# Clash 配置说明

本目录提供两套主配置：

- `通用版.yaml`
- `openclash_compatible.yaml`

## 客户端适配建议

- `Clash Verge`：使用 `通用版.yaml`
- `FlClash`：使用 `通用版.yaml`
- `luci-app-fchomo`：使用 `通用版.yaml`
- `luci-app-momo`：使用 `通用版.yaml`
- `luci-app-nikki`：使用 `通用版.yaml`
- `luci-app-openclash`：
  - Meta(Mihomo) 内核：`通用版.yaml`
  - Premium 内核：`openclash_compatible.yaml`

## 为什么有两个版本

- `通用版.yaml` 使用了 `mrs` 规则格式，适合 Mihomo 系客户端。
- `openclash_compatible.yaml` 使用 `text/yaml` 规则格式，兼容 OpenClash Premium 内核。

## 使用步骤

1. 导入配置文件。
2. 将 `订阅1/订阅2/订阅3` 替换为你自己的订阅地址。
3. OpenClash 用户确认内核类型后再选对应配置。

## 常见问题

- OpenClash 提示规则不支持：
  - 说明你可能在 Premium 内核加载了 `通用版.yaml`，请改用 `openclash_compatible.yaml`。
- 无节点：
  - 检查订阅链接是否已替换、是否可访问。
