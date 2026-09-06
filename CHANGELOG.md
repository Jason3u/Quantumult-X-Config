# 修改日志（Changelog）

> 本文件记录 qx-public.conf 配置的历次修改。每次修改前请先阅读本文件了解当前状态，修改后必须在此追加记录。

## 2026-09-06（四）· Binance 默认出口改香港自动

### 排查结论（大陆网络实测）
用户反馈「Binance 操作不顺畅，怀疑有时走了代理」。实测（大陆出口）结论与直觉相反——**问题不是走了代理，而是 direct 默认已是死路**：
- 规则完整度无问题：16 条主域/镜像域/CDN/链上浏览器，为 blackmatrix7 规则（12 条）完整超集
- **主域直连全部被墙**：api/accounts/www.binance.com、stream.binance.com:9443（行情 WebSocket）、cdn-apps.binance.com、public.bnbstatic.com（静态 CDN）直连全部超时
- **大陆镜像域基本失效**：binancecnt.com / binancezh.com / bnappzh.co / binance.me / binance.cloud 全超时，仅 bnbzh.ac 可通（2.9s，慢）
- 由此产生的实际体验：App 每次命中主域的请求都要经历 5 秒级超时+重试，再靠残存镜像或缓存走通 → 「每个操作都转圈」；且部分未覆盖埋点域名走兜底（香港），App 出口 IP 在大陆/香港间跳变，可能触发币安风控摩擦

### 修改
- **Binance 组默认 `direct` → `香港自动`**：`static=Binance, 香港自动, 香港住宅IP, 香港节点, direct, 日本节点, 新加坡节点, 台湾节点, 韩国节点`
- direct 保留为手动选项（列于香港选项之后），想切回随时可在策略组里手动选
- Clash 配置（ClashForSelf.yaml）同步修改（DIRECT 同样保留为手动选项）
- 分流规则文件（qx/Binance.list / clash/Binance.yaml）无需改动，覆盖度已完备

## 2026-09-06（三）· 兜底分流重排

### 修改
- **兜底分流仅保留四个地区并重排**：`香港自动, 台湾节点, 韩国节点, 日本节点`（顺序即优先级，默认香港自动=住宅优先/无住宅回退香港）
- **移出未提到的选项**：自动选择 / 美国节点 / 新加坡节点 / 英国节点 / direct / proxy 全部从兜底分流移除
- Clash 配置（ClashForSelf.yaml）同步修改（含移出 DIRECT / 手动切换）

## 2026-09-06（二）· Cornix 延迟排查

### 结论
Cornix「每个界面点击都转圈」的两个配置侧原因：
1. ~~测速组 `tolerance=0` 频繁切换节点，WebSocket（`wss://dashboard.cornix.io/ws/ws/`）反复断连~~ —— 上一次修复已解决
2. **Cornix 分流规则不完善**：原规则仅 `cornix.io` 一条。实测 dashboard JS 包，其第三方运行时依赖（intercom 客服组件 / country.is 国家检测 / mixpanel·mxpnl·avo 埋点 / hotjar / sentry 错误上报 / whop 支付 / calendly / googletagmanager）全部漏到兜底分流，每个域名还要先经国内 DNS 解析再判断 geoip，且出口与 Cornix 组不一致 —— 已在规则仓库补全（`qx/Cornix.list`，12 条），QX 侧 update-interval=3600 到期自动拉新，**本配置文件无需改动**

### 备注
- 剩余不可避免的延迟：香港节点 → Cornix 后端（欧美）单程 200ms+ 物理延迟，每次点击含多个 API 往返，1 秒左右转圈属于正常范围
- Clash 配置同步：url-test 组 `tolerance: 0` → `100` 已修复（见 Clash-config CHANGELOG）

## 2026-09-06 · 连接卡顿排查修复

### 排查背景
- 症状：所有 App 持续转圈刷新、消息接收延迟（国内国外 App 均受影响）
- 实测：订阅有效（25 节点 / 剩 447.62 GB / 2026-10-04 到期）；节点全部 VLESS（14 个 Reality + 11 个 TLS/WS）；**机场当前无「住宅」节点**（香港住宅IP 组为空，不影响「香港自动」正则回退到香港节点）；规则源与解析器 URL 服务端均正常
- 已确认无问题：QX ≥ 1.6.0（Reality 兼容）、MitM 证书已安装信任、节点延迟测试多数正常

### 修改
- **所有 url-latency-benchmark 组 `tolerance=0` → `tolerance=100`**：零容差导致任何 1ms 波动即切换节点，IP 频繁跳变，消息类 App 频繁断连重连（接收延迟的直接嫌疑之一）
- **[dns] 新增 `doh-server=https://doh.pub/dns-query`**：`geoip, cn, direct` 规则要求每个新连接先本地解析一次；国内明文 UDP 53 可能被运营商劫持/丢包，拖慢所有 App（含直连的国内 App）。DoH 加密防劫持，与原 UDP DNS 并发竞速

### 待二分定位（用户侧）
1. 重新下载配置后若仍卡：设置 → MitM → 关闭总开关，用 10 分钟。恢复 → 重写脚本污染响应（头号嫌疑：APP解锁合集 Collections.conf，其 MitM 主机名覆盖大量 App；其次微信相关两条）
2. 关 MitM 仍卡：完全关闭 QX 测国内 App，还卡则是网络/运营商问题，与配置无关

### 备注
- 订阅 `opt-parser=true` 依赖 fastly.jsdelivr.net 的解析器脚本，jsdelivr 大陆不稳时订阅更新会失败（排查期间保持不动，避免引入变量；主因排除后再评估去掉）
- Clash 配置（ClashForSelf.yaml）的 url-test 组同样存在 `tolerance: 0` 问题，待 QX 侧定位后同步修复

## 2026-09-06

### 策略组
- **新增 TradingView 策略组**：默认「香港自动」（住宅优先，无住宅回退香港节点），备选香港住宅IP / 香港 / 日本 / 新加坡 / 台湾 / 韩国，可手动切换

### 分流规则
- **新增 TradingView 分流规则**（自建，[Proxy-Rules-Collection](https://github.com/Jason3u/Proxy-Rules-Collection) `qx/TradingView.list`）：覆盖 `tradingview.com` 主域（官网 / 中文站 / 图表数据 / 静态资源 / API），指向 TradingView 策略组，update-interval=3600
- 与 Clash 配置（ClashForSelf.yaml）同步新增，两端分流行为保持一致

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
