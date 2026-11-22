# wenyi_clash_config

个人 Mihomo / OpenClash 分流配置，使用 **Sub-Store 组合订阅 + 纯分流覆写模板** 生成最终配置。

## 文件

- `mihomo-override.yaml`：包含 `profile.store-selected`、`proxy-groups`、`rule-providers`、`rules` 的分流覆写模板，不是独立节点订阅。
- `rules/claude.list`：Claude / Anthropic 高覆盖率规则，按“宁可错杀，不可放过”维护。
- `rules/gemini-extra.list`：补充 Gemini、AI Studio、NotebookLM、Jules、Antigravity、Cloud Code 等 Google AI 端点。

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

1. **Claude**：本仓库高覆盖率规则（核心域名、登录 / API / MCP / CDN、KYC、遥测、客服、Claude Code 插件与 Artifact 依赖、静态 CIDR）；完全不使用 IP-ASN 或 ASN.mmdb。
2. **Gemini**：blackmatrix7 Gemini 规则 + 本仓库维护的 Google AI / Antigravity 补充规则，整合 VPSDance/ai-proxy-rules 的 Google AI provider（Gemini、AI Studio、NotebookLM、Jules、Antigravity、Cloud Code、Labs、Flow、Opal、Stitch 等端点）。
3. **AI**：OpenAI、ChatGPT、Copilot，以及 clash-rules-cn / Sukka 的综合 AI 规则。

Claude 规则有意采取主动过匹配：除 Anthropic / Claude / MCP 核心域名外，还纳入 Persona KYC、Google Maps、Sentry、Statsig、Datadog、Sift、Intercom、Fathom，以及 Claude Code 官方网络要求中的 Google Storage、npm Registry、GitHub、raw.githubusercontent.com、Gerrit、Homebrew 和 Artifact 字体 / CDN 依赖。这些第三方服务可能被其他应用使用，因此会有意把部分非 Claude 流量一起送入 Claude 策略组；GitHub / GitHubusercontent、npmjs.org 和 googlesource.com 采用后缀匹配，覆盖动态子域名，误匹配范围会更大。这是按「宁可错杀，不可放过」原则作出的取舍。

MCP 服务端域名由用户或第三方自行定义，无法仅靠域名规则识别所有不含 `mcp` 关键词的任意 MCP 主机；如果实际使用的 MCP 服务域名不含相关关键词，需要按实际域名补充到 `rules/claude.list`。

### Claude IP 范围（不使用 ASN）

所有 IP 规则已集中放入 `rules/claude.list`，由现有 `RULE-SET,claude` 统一送入 Claude 策略组。配置不使用 `IP-ASN`，也不需要 `ASN.mmdb`。

Anthropic 官方公布的入站地址段为：

- IPv4：`160.79.104.0/23`
- IPv4：`153.61.224.0/22`
- IPv6：`2607:6bc0::/48`

此外，BGP 路由记录显示 Anthropic 的 AS399358 还公告了独立的 IPv6 前缀 `2607:6bc0:11::/48`；BGP.HE 将其 Origin 标记为 AS399358（Anthropic, PBC），并显示 ROA 签名有效。它没有列在 Anthropic 面向 API 用户的固定 IP 文档中，但有独立的 BGP / ASN 归属证据，因此也作为静态规则纳入：

- `2607:6bc0:11::/48`：AS399358 公告的 IPv6 前缀；来源：[BGP.HE 前缀记录](https://bgp.he.net/net/2607%3A6bc0%3A11%3A%3A/48)、[AS399358 公告列表](https://bgp.he.net/AS399358)。

为贯彻高覆盖率原则，同时参考较宽的地址范围，实际规则还保留：

- `160.79.104.0/21`：比官方入站 `160.79.104.0/23` 更宽，包含该官方网段；Anthropic 也将这个 `/21` 列为出站范围，因此这里有意连同该地址块一起过匹配。
- `153.61.224.0/22`：官方入站网段。
- `2607:6bc0::/32`：比官方入站 `2607:6bc0::/48` 更宽，包含该官方网段，也覆盖上面明确列出的 `2607:6bc0:11::/48`。

因此，`2607:6bc0:11::/48` 在匹配范围上已被 `2607:6bc0::/32` 包含；仍单独列出是为了保留 BGP 公告来源和明确归属记录。这里使用的是静态 CIDR，不会恢复 `IP-ASN` 或 `ASN.mmdb` 依赖。

宽网段会有意覆盖同一地址范围内可能与 Claude 无关的 IP 流量。官方出站网段 `153.61.208.0/20` 不加入客户端目标规则，因为它用于 Anthropic 向外部 MCP 等服务发起连接，不是用户访问 Claude API 的目标地址。

## OpenClash「绕过中国大陆」

本模板不再加载 `china-ip` / `domestic-ip` 规则集，也不写 `GEOIP,CN,DIRECT`。如果 OpenClash 已启用「绕过中国大陆」，保持开启即可。该功能会在插件层把符合条件的大陆目标 IP 流量绕过内核；因此本模板不重复维护 `china-ip` / `domestic-ip`，也不写 `GEOIP,CN,DIRECT`。

模板仍保留国内常见软件域名、国内直连域名、Apple CDN、Apple CN 等域名规则。局域网 / 私有地址规则也保留，因为它们不是大陆 IP 绕过的替代品。

如果某个 Claude 域名恰好被 OpenClash 的大陆绕过提前放行、导致它没有进入 Mihomo 规则引擎，可以把该域名加入 OpenClash「绕过中国大陆 IPv4 黑名单」，使它重新进入内核，再由本模板的 Claude 规则处理。黑名单中的域名 / IP 会正常进入 Clash 按规则分流，参见 [OpenClash 设置说明](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenClash-%E8%AE%BE%E7%BD%AE%E6%96%B9%E6%A1%88)。

## 规则来源

- [clash-rules-cn](https://github.com/mcxiaochenn/clash-rules-cn)：国内直连、常见软件、AI、GFWList、广告、私有域名等聚合规则。
- [Sukka Ruleset](https://ruleset.skk.moe/)：AI、Apple、Microsoft、流媒体、国内直连、全球直连、广告、下载等规则。
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)：Gemini / OpenAI / Copilot、YouTube、Netflix、Telegram、Apple、Microsoft、Game、GitHub 等应用规则；Claude 使用本仓库的高覆盖率专属规则。
- [VPSDance/ai-proxy-rules](https://github.com/VPSDance/ai-proxy-rules)：Google AI provider 的 Gemini、AI Studio、NotebookLM、Jules、Antigravity、Cloud Code、Labs、Flow、Opal、Stitch 等域名规则；本仓库将域名规则整合进 Gemini 补充列表，不使用其 PROCESS-NAME 规则。

## 关于旧模板中的节点 URL

旧版 `Merge.yaml` 曾把两个订阅 URL 直接写在公开仓库中。新版已移除这层硬编码，节点统一由 Sub-Store 的源订阅提供。若旧 URL 仍有效，建议视为已公开暴露并在对应服务商处轮换订阅链接。


## 节点选择与其他分组

- `♻️ 自动选择`：对除 Oracle 外的全部节点做 URL 测速，自动选择延迟较低的节点。
- `🌐 手动切换`：按国家进入对应国家组；每个国家组默认走自己的自动测速子组，也可以直接选择该国任意节点。
- `☁️ 甲骨文圣何塞`：只精确筛选最终 YAML 中名为 `oracle-vless-reality` 和 `oracle-hy2-hop` 的两个自建节点；不兼容旧名称，不参加美国筛选、国家组和全局自动测速。本次国家组改造不修改 Oracle 策略组。
- `📺 YouTube`：单独分组。
- `🎬 流媒体`：统一承接 Netflix 和其他流媒体；Netflix 的专属规则集仍保留用于精准识别，但不再单独建立策略组。
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

- Claude 和 Gemini：使用 [Lobe Icons](https://github.com/lobehub/lobe-icons) 的彩色 SVG CDN。
  - Claude：`https://unpkg.com/@lobehub/icons-static-svg@latest/icons/claude-color.svg`
  - Gemini：`https://unpkg.com/@lobehub/icons-static-svg@latest/icons/gemini-color.svg`
- 流媒体、游戏平台、文件下载：使用 [Koolson/Qure](https://github.com/Koolson/Qure) 的语义化彩色图标，避免把 Netflix / Steam 品牌图标误当成整个分类。
  - 流媒体：`https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Streaming.png`
  - 游戏平台：`https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Game.png`
  - 文件下载：`https://raw.githubusercontent.com/Koolson/Qure/master/IconSet/Color/Download.png`
