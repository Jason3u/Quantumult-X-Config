# Quantumult X Config · Jason 自用

个人 Quantumult X 分流配置，含节点策略组、远程规则、复写脚本等，仅供自用参考。

> 配置修改记录见 [CHANGELOG.md](CHANGELOG.md)，每次修改前请先阅读。

## 使用方法

1. 复制配置文件 raw 地址：
   `https://raw.githubusercontent.com/Jason3u/Quantumult-X-Config/main/qx-public.conf`
   （或 jsdelivr 镜像：`https://cdn.jsdelivr.net/gh/Jason3u/Quantumult-X-Config@main/qx-public.conf`）
2. Quantumult X：风车 → 配置文件 → 下载，粘贴地址下载并启用
3. 配置内已含机场订阅（server_remote），更新订阅后即可使用

## 分流总览

| 流量类型 | 策略 | 说明 |
| --- | --- | --- |
| Apple 核心（AppStore/iCloud/AppleMusic 等） | direct | 直连 |
| Apple 外区（AppleNews） | 兜底分流 | 走代理 |
| AppleTV | InternationalStreaming | 走国际流媒体策略组 |
| OpenAI / Gemini / Claude / Grok | 各自独立策略组 | 手动选地区，规避 IP 跳变风控 |
| X（Twitter） | X | 默认香港自动，可手动切地区 |
| Binance 币安 | Binance | 默认国内直连，可手动切友好地区 |
| OKX / Bybit / Bitget / Gate | 各自独立策略组 | 默认「香港自动」 |
| TradingView 行情图表 | TradingView | 默认香港自动，可手动切地区 |
| Cornix 自动交易 | Cornix | 默认香港节点，可手动切新加坡/日本 |
| Fomo | Fomo | 默认 direct 直连；可手动切换地区节点 |
| 国际流媒体（Netflix/Disney+/HBO/YouTube/Spotify/TikTok 等） | InternationalStreaming | 默认香港节点，可手动切地区 |
| 国内流量 | direct | geoip cn 直连 |
| 其余外网 | 兜底分流 | 各地区测速组自动选最低延迟 |

## 香港自动机制

```
available=香港自动, server-tag-regex=住宅|香港
```

- `available` 组用正则直接匹配节点，自动选用第一个可用节点
- 机场有住宅 IP 节点时优先住宅，没有时自动回退普通香港节点，换机场无需改配置
- 注意：QX 的 `available` **不支持嵌套其它策略组**，只能直接匹配节点（嵌套空组会导致 REJECT）
- 各交易所 static 组保留「香港住宅IP」选项，需要时可手动强制住宅

## 自建规则

- 规则仓库：[Jason3u/Proxy-Rules-Collection](https://github.com/Jason3u/Proxy-Rules-Collection)
- QX 规则位于 `qx/` 目录：X / Binance / OKX / Bybit / Bitget / Gate / TradingView / Cornix / Fomo 的 `.list` 文件
- 通过 jsdelivr CDN 分发，每小时自动更新（update-interval=3600）

## 注意事项

- 配置内含机场订阅 token 与 MitM 证书（p12），**仅供个人使用**，请勿公开传播
- 策略与规则按个人风控需求定制（交易所住宅 IP、AI 固定地区等），照抄请结合自身机场情况调整
- Fomo 的地区可用性和永续合约资格以官方条款为准；策略组排序仅代表网络出口偏好，不用于规避地区限制
