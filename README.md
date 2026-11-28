# wenyi_clash_config

个人 Mihomo / OpenClash 分流配置，使用 **Sub-Store 组合订阅 + 纯分流覆写模板** 生成最终配置。

## 文件

- `mihomo-override.yaml`：包含 `profile.store-selected`、`proxy-groups`、`rule-providers`、`rules` 的分流覆写模板，不是独立节点订阅。
- `rules/claude.list`：Claude 专属域名补充及旧订阅兼容规则。
- `rules/gemini-extra.list`：补充 Gemini、AI Studio、NotebookLM、Jules、Antigravity、Cloud Code 等 Google AI 端点。
- `rules/longbridge.list`：长桥桌面端、官方服务、HTTP API、行情与交易 WebSocket 域名。

## 配置边界：只管分流，不接管运行环境

本模板只包含四个顶层区块：

- `profile.store-selected`：持久化策略组选择，重启后保留用户手动选择。
- `proxy-groups`：策略组。
- `rule-providers`：远程域名 / IP 规则集。
- `rules`：规则匹配顺序和策略组指向。

模板不包含也不覆盖以下配置：

- `dns`、Fake-IP、Fake-IP Filter、nameserver、fallback、nameserver-policy。
- DNS 劫持、DNS 分流、DNS 上游、Hosts。
- `tun`、`sniffer`、端口监听、mixed-port、allow-lan。
- `mode`、`log-level`、IPv6、Fake-IP 持久化（`profile.store-fake-ip`）。
- QUIC、TCP 并发、指纹、进程匹配等运行参数。

除 `profile.store-selected` 仅用于保存策略组选择外，其余运行环境功能全部交给 OpenClash 自身设置和 Meta 内核管理。本模板不会写入第二套 DNS / Fake-IP / 嗅探配置，避免与 OpenClash 的插件设置重复或互相覆盖。

## Sub-Store 使用流程

1. 在 Sub-Store 中准备一个源订阅或组合订阅，确保其中包含你需要的机场节点，以及 Oracle 两个节点。
2. 你已在 3X-UI 上游将两个节点名称改为 `oracle-vless-reality` / `oracle-hy2-hop`。刷新 Sub-Store 上游订阅后，检查最终 YAML 的 `proxies[].name` 是否同步为这两个名称。策略组只精确匹配这两个完整名称；旧名称、中文别名或其他仅包含 `oracle` 的节点都不会入组。注意：Sub-Store 列表中的显示名称不一定等于最终 YAML 的 `proxies[].name`，应以最终 YAML 为准。
3. 在 Sub-Store 的「文件」中创建 Mihomo 配置，**Source 选择包含机场和甲骨文节点的 Collection，并启用 `Add Proxies`（添加节点）**，确保集合里的节点实际写入最终配置的 `proxies:` 区块；然后使用远程覆写。
4. 远程覆写 URL：

   `https://raw.githubusercontent.com/Hwenyi/wenyi_clash_config/main/mihomo-override.yaml`



如果甲骨文策略组仍为空，先打开 Sub-Store 最新生成的最终 YAML，检查 `proxies[].name`。必须分别是 `oracle-vless-reality` 和 `oracle-hy2-hop`；如果仍然显示 `reality` / `hy2-hop` 或其他名称，精确筛选会故意不匹配，先刷新 3X-UI 上游订阅并重新生成。若最终 YAML 已是两个指定名称但策略组仍为空，再确认远程覆写已刷新且 OpenClash 已重新加载最新配置。策略组显示 `COMPATIBLE` 时，表示筛选后没有成员；这是 Mihomo 对空策略组的默认回退值，不代表甲骨文节点已经被选中。

5. 使用 Sub-Store 生成的**最终配置文件 URL**导入 OpenClash。不要把本仓库的覆写文件 URL 直接当作完整订阅导入。

策略组使用 Mihomo 的 `include-all: true` 从 Sub-Store 最终配置中的节点集合读取节点，不在模板里再写一套机场订阅 URL。Oracle 组只精确匹配最终 YAML 中完整名称 `oracle-vless-reality` / `oracle-hy2-hop`；全局自动选择、全部节点、各国家自动测速子组和国家选择组均按同一组名称排除这两个节点。

## Oracle 节点

`☁️ 甲骨文圣何塞` 只按 Sub-Store 最终生成 YAML 中的 `proxies[].name` 精确匹配以下两个完整名称：

| 位置 | VLESS Reality | Hysteria2 |
|---|---|---|
| 3X-UI 上游节点名称 | `oracle-vless-reality` | `oracle-hy2-hop` |
| 最终 YAML 必须出现的 `proxies[].name` | `oracle-vless-reality` | `oracle-hy2-hop` |

筛选正则为 `(?i)^oracle-(vless-reality|hy2-hop)$`。它只接受这两个完整节点名（大小写不敏感），不接受旧名称、中文别名或其他 Oracle 节点名，也不会因为其他节点名称中包含 `oracle` 就将其纳入。若上游改名后最终 YAML 没有同步更新，策略组将保持空组；以最终 YAML 的 `proxies[].name` 为准排查。

## 国家节点筛选

每个国家只有一个对用户展示的选择组，名称就是国家本身（例如 `🇭🇰 香港`、`🇯🇵 日本`），不再使用「自动 / 手动」后缀。每个国家组内部有一个带对应国旗的「自动选择」入口，由独立的隐藏 `url-test` 子组测速；同时通过 `include-all` 列出该国全部节点，允许直接手动覆盖。通过 `default-selected` 明确把自动入口设为默认项；手动选择具体节点后不会被自动测速抢回。`profile.store-selected: true` 会保存选择，重启后继续使用。全局 `♻️ 自动选择` 保持原样，继续对除甲骨文外的全部机场节点测速。甲骨文不参与这次国家组改造。美国除国家名、旗帜、US/USA 外，还识别洛杉矶、圣何塞、圣克拉拉、旧金山、芝加哥、达拉斯、西雅图、波特兰、纽约、阿什本、弗吉尼亚等常见机房城市。

筛选正则使用 Go / RE2 兼容语法，不使用 Lookahead / Lookbehind。

## AI 分流优先级

1. **Claude**：MetaCubeX Anthropic 域名规则、Claude 专属补充及 `IP-ASN,399358`（Anthropic, PBC）；IP 归属由 Mihomo 的 ASN 数据库识别。
2. **Gemini**：blackmatrix7 Gemini 规则 + 本仓库维护的 Google AI / Antigravity 补充规则，整合 VPSDance/ai-proxy-rules 的 Google AI provider（Gemini、AI Studio、NotebookLM、Jules、Antigravity、Cloud Code、Labs、Flow、Opal、Stitch 等端点）。
3. **AI**：OpenAI、ChatGPT、Copilot，以及 clash-rules-cn / Sukka 的综合 AI 规则。

Claude 复用 [MetaCubeX 的 Anthropic 域名规则](https://github.com/MetaCubeX/meta-rules-dat/blob/meta/geo/geosite/anthropic.list)。`rules/claude.list` 为已有订阅保留原路径，并补充 Claude 专属 CDN、身份验证及 MCP 协议站点。共享仓库、包管理器、字体/CDN 和第三方遥测服务按常规规则分流。自定义 MCP 服务的域名由其运营者决定，仅能按实际域名分流。

GitHub 网页、API、SSH、Release 下载及 GHCR 复用 blackmatrix7 的 GitHub 规则。`formulae.brew.sh` 复用 [MetaCubeX 的 Homebrew 规则](https://github.com/MetaCubeX/meta-rules-dat/blob/meta/geo/geosite/homebrew.list)。两份规则都进入 `⌨️ GitHub` 策略组。

### Claude IP 匹配（AS399358）

Claude 的 IP 级补充匹配恢复为：

```yaml
IP-ASN,399358,Claude,no-resolve
```

AS399358 的注册主体为 Anthropic, PBC。此前通过 BGP 路由公告核实，`2607:6bc0:11::/48` 的 Origin ASN 为 AS399358，ROA 状态有效。恢复 ASN 匹配后，Mihomo 会按目标 IP 的 ASN 归属匹配 Anthropic 公告的地址，而不是依赖手工维护的 CIDR 列表。

因此，`rules/claude.list` 只维护域名规则；Claude 的 IP 级匹配单独放在主 `rules:` 中，避免将 `IP-ASN` 混入域名 Provider。该规则需要可用的 ASN 数据库（`ASN.mmdb`），OpenClash 小闪存模式可将相关数据放在内存中。
## 长桥证券分流

`💹 长桥证券` 使用独立域名规则，将长桥桌面端及其服务域名统一送入可手动选择的策略组。域名清单依据 [V2Fly Domain List Community 的 Longbridge 列表](https://github.com/v2fly/domain-list-community/blob/master/data/longbridge)，覆盖 `longbridge.cloud`、`longbridge.cn`、`longbridge.com`、`longbridge.global`、`longbridge.hk`、`longbridge.sg`、`longbridgeapp.com`、`longbridgehk.com`、`longportapp.com`、`lbctrl.com`、`lbkrs.com`、`wbrks.com`。

长桥官方 OpenAPI 文档列出的 HTTP API、行情 WebSocket、交易 WebSocket 端点包括 `openapi.longbridge.com`、`openapi-quote.longbridge.com`、`openapi-trade.longbridge.com`、`openapi.longbridge.cn`、`openapi-quote.longbridge.cn`、`openapi-trade.longbridge.cn`；这些主机均由 `longbridge.com` / `longbridge.cn` 后缀规则覆盖，见[官方接入点文档](https://open.longbridge.com/docs/getting-started)及[官方 WebSocket 端点文档](https://open.longbridge.com/docs/socket/hosts)。

该策略组默认使用 `🎯 节点选择`，可单独切换到任一国家组、甲骨文圣何塞、全部节点或 DIRECT。规则使用域名匹配，不依赖 `PROCESS-NAME`：OpenClash 部署在路由器上时，无法可靠识别其他 LAN 设备内部的桌面进程。

## OpenClash「绕过中国大陆」

本模板不再加载 `china-ip` / `domestic-ip` 规则集，也不写 `GEOIP,CN,DIRECT`。如果 OpenClash 已启用「绕过中国大陆」，保持开启即可。该功能会在插件层把符合条件的大陆目标 IP 流量绕过内核；因此本模板不重复维护 `china-ip` / `domestic-ip`，也不写 `GEOIP,CN,DIRECT`。

模板仍保留国内常见软件域名、国内直连域名、Apple CDN、Apple CN 等域名规则。局域网 / 私有地址规则也保留，因为它们不是大陆 IP 绕过的替代品。

如果某个 Claude 域名恰好被 OpenClash 的大陆绕过提前放行、导致它没有进入 Mihomo 规则引擎，可以把该域名加入 OpenClash「绕过中国大陆 IPv4 黑名单」，使它重新进入内核，再由本模板的 Claude 规则处理。黑名单中的域名 / IP 会正常进入 Clash 按规则分流，参见 [OpenClash 设置说明](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenClash-%E8%AE%BE%E7%BD%AE%E6%96%B9%E6%A1%88)。

## 规则来源

- [clash-rules-cn](https://github.com/mcxiaochenn/clash-rules-cn)：国内直连、常见软件、AI、GFWList、广告、私有域名等聚合规则。
- [Sukka Ruleset](https://ruleset.skk.moe/)：AI、Apple、Microsoft、流媒体、国内直连、全球直连、广告、下载等规则。
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)：Gemini / OpenAI / Copilot、YouTube、Netflix、Telegram、Apple、Microsoft、Game、GitHub 等应用规则。
- [V2Fly Domain List Community](https://github.com/v2fly/domain-list-community/blob/master/data/longbridge)：提供长桥官方域名与站点资源域名清单。
- [VPSDance/ai-proxy-rules](https://github.com/VPSDance/ai-proxy-rules)：Google AI provider 提供 Gemini、AI Studio、NotebookLM、Jules、Antigravity、Cloud Code、Labs、Flow、Opal、Stitch 等域名规则。本仓库将 Google AI 域名规则整合进 Gemini 补充列表，不使用其 `PROCESS-NAME` 规则。

## 关于旧模板中的节点 URL

旧版 `Merge.yaml` 曾把两个订阅 URL 直接写在公开仓库中。新版已移除这层硬编码，节点统一由 Sub-Store 的源订阅提供。若旧 URL 仍有效，建议视为已公开暴露并在对应服务商处轮换订阅链接。


## 节点选择与其他分组

- 除 `🌍 全球直连` 和 `🛑 广告拦截` 外，所有面向流量的业务策略组及 `🐟 漏网之鱼` 均提供完整出口选项：14 个国家组、甲骨文圣何塞、全部节点、手动切换、节点选择、自动选择及 DIRECT。苹果服务、微软服务继续以 DIRECT 为默认出口；其余业务组沿用节点选择作为默认出口。

- `♻️ 自动选择`：对除 Oracle 外的全部节点做 URL 测速，自动选择延迟较低的节点。
- `🌐 手动切换`：按国家进入对应国家组；每个国家组默认走自己的自动测速子组，也可以直接选择该国任意节点。
- `☁️ 甲骨文圣何塞`：只精确筛选最终 YAML 中名为 `oracle-vless-reality` 和 `oracle-hy2-hop` 的两个自建节点；不兼容旧名称，不参加美国筛选、国家组和全局自动测速。本次国家组改造不修改 Oracle 策略组。
- `💹 长桥证券`：按长桥专属域名规则分流，默认走节点选择，可切换任意国家组。
- `🎬 流媒体`：统一承接 YouTube、Netflix、Disney+、Spotify、TikTok、Twitch 等流媒体服务。YouTube / Netflix 保留 blackmatrix7 的专属规则集；其他流媒体使用 Sukka 的 `stream.txt` / `stream-ip.txt` 聚合规则。所有规则共用这一个策略组，不再拆分独立组。
- `🌍 全球直连`：对应 Sukka 的 `direct.txt`，是“指定不代理”的规则集合，不代表所有海外网站都直连。它包含本地域名 / 路由器域名、部分 PT 站点、学术与文献站点，以及少量进程规则；默认走 `DIRECT`，也允许手动切到代理。
  - 这组的价值是给这些特定目标保留一个统一的手动出口：平时直连，遇到特定网络环境直连失败时可以临时切代理。
  - 你已经有更靠前的局域网、私有地址和国内域名直连规则；这些规则会先命中，不会依赖「全球直连」组。
  - Sukka 的 `direct.txt` 还包含 `PROCESS-NAME` 条目。OpenClash 运行在路由器上时，通常无法识别其他 LAN 设备内部的应用进程；这部分不能当作手机 / 电脑应用级分流，域名规则才是主要作用。
- 国内软件、国内域名、Apple / Microsoft CDN 等指定规则直接指向 `DIRECT`，不再额外设置「国内直连」策略组。
- `🛑 广告拦截`：默认 REJECT，可在组内切换 REJECT-DROP、DIRECT 或节点选择。
 
## 漏网之鱼的工作原理

`🐟 漏网之鱼` 不是额外的域名清单，而是最后一条 `MATCH` 规则。前面所有规则都没有命中的连接，才会落到这里。默认走 `🎯 节点选择`，也可以手动切成 DIRECT。它的作用是给未知流量一个明确、可控的兜底出口，避免流量因为没有匹配规则而行为不确定。

「全球直连」与「漏网之鱼」不是一回事：前者只处理 Sukka `direct.txt` 中明确列出的目标；后者处理所有前置规则都没有匹配到的剩余流量。前者是一个有限清单，后者是最后的兜底规则。


## OpenClash DNS / Fake-IP 设置

OpenClash 的运行模式、DNS 劫持、Dnsmasq 转发、DNS 分流、嗅探、QUIC 禁用和大陆 IP 绕过，均由 OpenClash 插件设置管理。使用本模板时，不需要在模板中再配置一遍，也不需要为了本模板额外开启 Fake-IP Filter、Nameserver-Policy 或自定义上游 DNS。

如果你已经在 OpenClash 中配置好 DNS 和嗅探，保持现有设置即可。模板没有 `dns` / `sniffer` / `tun` 区块，不会主动覆盖这些选项。


## 策略组图标

- 仅 Claude 和 Gemini 保留 Lobe Icons 彩色 PNG 图标，以补充其策略组名称。
  - Claude：`https://unpkg.com/@lobehub/icons-static-png@latest/dark/claude-color.png`
  - Gemini：`https://unpkg.com/@lobehub/icons-static-png@latest/dark/gemini-color.png`
- 其他策略组不设置 `icon` URL，避免远程图标与策略组名称中已有的 Emoji 重复显示。
