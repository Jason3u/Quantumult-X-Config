# 修改日志（Changelog）

> 本文件记录 qx-public.conf 配置的历次修改。每次修改前请先阅读本文件了解当前状态，修改后必须在此追加记录。

## 2026-09-04

### 配置结构
- **自建规则仓库迁移**：规则地址从 `Jason3u/Quantumult-X-Rules` 更新为 `Jason3u/Proxy-Rules-Collection`（旧仓库已失效，规则文件位于新仓库 `qx/` 目录）
- **机场订阅**：更换为「吹雪云」
- **新增「香港自动」策略组**：`available=香港自动, server-tag-regex=住宅|香港`。有住宅 IP 节点优先住宅，无住宅时自动回退香港节点，换机场无需改配置
  - ⚠️ 经验教训：QX 的 `available` 组**不支持嵌套其它策略组**，只能用 `server-tag-regex` 直接匹配节点。嵌套空组不会回退，会导致 REJECT
- **新增 InternationalStreaming 国际流媒体策略组**：规则来源 ddgksf2013/Filter 的 Streaming.list，已存入自建仓库 `qx/InternationalStreaming.list`；默认香港节点，备选按风控严重性排序（最严重排最后）；规则启用 `opt-parser=true` 资源解析器
- **AppleTV**：从兜底分流改走 InternationalStreaming
- **X 策略组**：从单一「香港节点」改为「香港自动 + 多地区备选」，避免香港全挂时断网

### 策略组当前状态
| 策略组 | 默认 | 备注 |
| --- | --- | --- |
| OKX / Bybit / Bitget / Gate | 香港自动 | 可手动强制香港住宅IP |
| Binance | direct | 可手动切友好地区 |
| Cornix | 香港节点 | 可手动切新加坡/日本 |
| Fomo | direct | 可手动切地区节点 |
| X | 香港自动 | 可手动切地区 |
| InternationalStreaming | 香港节点 | 备选按风控严重性排序：台湾→新加坡→韩国→日本→英国→美国 |

### 文档
- 新增本仓库 README.md 说明文件
