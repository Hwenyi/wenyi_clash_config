# wenyi_clash_config

个人 Mihomo / OpenClash 分流配置，使用 **Sub-Store 组合订阅 + 纯分流覆写模板** 生成最终配置。

## 文件

- `mihomo-override.yaml`：仅包含 `proxy-groups`、`rule-providers`、`rules` 三个顶层区块的分流模板，不是独立节点订阅。
- `rules/claude.list`：Claude / Anthropic 高覆盖率规则，按“宁可错杀，不可放过”维护。
- `rules/gemini-extra.list`：补充 Gemini、AI Studio、NotebookLM 等 Google AI 端点。

## 配置边界：只管分流，不接管运行环境

本模板严格只包含三个顶层区块：

- `proxy-groups`：策略组。
- `rule-providers`：远程域名 / IP 规则集。
- `rules`：规则匹配顺序和策略组指向。

模板不包含也不覆盖以下配置：

- `dns`、Fake-IP、Fake-IP Filter、nameserver、fallback、nameserver-policy。
- DNS 劫持、DNS 分流、DNS 上游、Hosts。
- `tun`、`sniffer`、端口监听、mixed-port、allow-lan。
- `mode`、`log-level`、IPv6、profile、Fake-IP 持久化。
- QUIC、TCP 并发、指纹、进程匹配等运行参数。

这些功能全部交给 OpenClash 自身设置和 Meta 内核管理。本模板不会再写入第二套 DNS / Fake-IP / 嗅探配置，避免与 OpenClash 的插件设置重复或互相覆盖。

## Sub-Store 使用流程

1. 在 Sub-Store 中准备一个源订阅或组合订阅，确保其中包含你需要的机场节点，以及 Oracle 两个节点。
2. 确认 Sub-Store 输出的两个甲骨文节点已加入最终配置。你这次提供的最终 YAML 中，两个节点的实际 `proxies[].name` 是 `reality` 和 `hy2-hop`；Sub-Store 订阅列表里显示的 `甲骨文-vless-reality` / `甲骨文-hy2-hop` 并不是最终节点名。策略组按最终 YAML 的真实节点名精确匹配，同时兼容旧的 Oracle / 中文别名。
3. 在 Sub-Store 的「文件」中创建 Mihomo 配置，**Source 选择包含机场和甲骨文节点的 Collection，并启用 `Add Proxies`（添加节点）**，确保集合里的节点实际写入最终配置的 `proxies:` 区块；然后使用远程覆写。
4. 远程覆写 URL：

   `https://raw.githubusercontent.com/Hwenyi/wenyi_clash_config/main/mihomo-override.yaml`



如果甲骨文策略组仍为空，先打开 Sub-Store 生成的最终 YAML，检查 `proxies[].name` 是否为 `reality` 和 `hy2-hop`。本次你提供的配置已经包含这两个节点，问题是此前策略组正则没有匹配这两个真实名称。若以后节点名变化，必须以最终 YAML 的 `proxies[].name` 为准调整筛选正则。策略组显示 `COMPATIBLE` 时，表示筛选后没有成员；这是 Mihomo 对空策略组的默认回退值，不代表甲骨文节点已经被选中。

5. 使用 Sub-Store 生成的**最终配置文件 URL**导入 OpenClash。不要把本仓库的覆写文件 URL 直接当作完整订阅导入。

策略组使用 Mihomo 的 `include-all: true` 从 Sub-Store 最终配置中的节点集合读取节点，不在模板里再写一套机场订阅 URL。Oracle 组精确匹配最终 YAML 中的 `reality` / `hy2-hop`，并兼容旧 Oracle / 中文别名；全局自动选择、全部节点和各国家手动组均排除这两个节点。

## Oracle 节点

`☁️ 甲骨文圣何塞` 必须按 Sub-Store 最终生成 YAML 中的 `proxies[].name` 匹配，而不是按订阅列表里的显示标签匹配：

| 位置 | VLESS Reality | Hysteria2 |
|---|---|---|
| Sub-Store 订阅列表显示 | `甲骨文-vless-reality` | `甲骨文-hy2-hop` |
| 最终 YAML 的实际节点名 | `reality` | `hy2-hop` |
| 节点协议 | `vless` | `hysteria2` |

当前策略组正则以 `^reality$` 和 `^hy2-hop$` 精确匹配这两个节点，同时兼容旧名称 `Oracle-vless-reality`、`Oracle-hy2-hop`、`甲骨文-vless-reality`、`甲骨文-hy2-hop` 及旧的圣何塞命名。锚定完整名称是为了避免误把其他含有 `reality` / `hy2-hop` 字样的机场节点收进来。

这次最终 YAML 已经包含两个节点，服务器地址均为你自己的 Oracle 主机；所以不是 Sub-Store 没有注入节点，而是之前的策略组正则没有匹配实际节点名。

## 国家节点筛选

国家分组只保留手动选择组，不再为每个国家单独创建自动测速组。全局 `♻️ 自动选择` 仍会对除甲骨文外的全部机场节点测速。美国除国家名、旗帜、US/USA 外，还识别洛杉矶、圣何塞、圣克拉拉、旧金山、芝加哥、达拉斯、西雅图、波特兰、纽约、阿什本、弗吉尼亚等常见机房城市。

筛选正则使用 Go / RE2 兼容语法，不使用 Lookahead / Lookbehind。

## AI 分流优先级

1. **Claude**：本仓库规则 + 官方 Anthropic 入站 IP + ASN。
2. **Gemini**：blackmatrix7 Gemini 规则 + 本仓库补充的 AI Studio、NotebookLM、Jules、Antigravity 等 Google AI 端点。
3. **AI**：OpenAI、ChatGPT、Copilot，以及 clash-rules-cn / Sukka 的综合 AI 规则。

Claude 规则有意把以下共享第三方服务也纳入 Claude：Sentry、Statsig、Datadog、Sift、Intercom、Fathom，以及包含 `mcp` 关键词的域名。这些服务也可能被其他应用使用，因此可能把其他应用的遥测或客服流量一起送入 Claude 策略组；这是本配置按用户要求采取的主动过匹配策略。

MCP 服务端域名由用户或第三方自行定义，无法仅靠域名规则识别所有不含 `mcp` 关键词的任意 MCP 主机；如果实际使用的 MCP 服务域名不含相关关键词，需要按实际域名补充到 `rules/claude.list`。

### Claude IP 范围

配置使用 Anthropic 官方公布的**入站**地址段：

- IPv4：`160.79.104.0/23`
- IPv4：`153.61.224.0/22`
- IPv6：`2607:6bc0::/48`

另参考 Net.Coffee 加入 `AS399358` 作为补充。没有把 Anthropic 官方文档中的出站网段当作客户端访问 Claude 的目标 IP：出站地址用于 Anthropic 向外部 MCP 服务发起请求，语义不同。

## OpenClash「绕过中国大陆」

本模板不再加载 `china-ip` / `domestic-ip` 规则集，也不写 `GEOIP,CN,DIRECT`。如果 OpenClash 已启用「绕过中国大陆」，保持开启即可。该功能会在插件层把符合条件的大陆目标 IP 流量绕过内核；因此本模板不重复维护 `china-ip` / `domestic-ip`，也不写 `GEOIP,CN,DIRECT`。

模板仍保留国内常见软件域名、国内直连域名、Apple CDN、Apple CN 等域名规则。局域网 / 私有地址规则也保留，因为它们不是大陆 IP 绕过的替代品。

如果某个 Claude 域名恰好被 OpenClash 的大陆绕过提前放行、导致它没有进入 Mihomo 规则引擎，可以把该域名加入 OpenClash「绕过中国大陆 IPv4 黑名单」，使它重新进入内核，再由本模板的 Claude 规则处理。黑名单中的域名 / IP 会正常进入 Clash 按规则分流，参见 [OpenClash 设置说明](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenClash-%E8%AE%BE%E7%BD%AE%E6%96%B9%E6%A1%88)。

## 规则来源

- [clash-rules-cn](https://github.com/mcxiaochenn/clash-rules-cn)：国内直连、常见软件、AI、GFWList、广告、私有域名等聚合规则。
- [Sukka Ruleset](https://ruleset.skk.moe/)：AI、Apple、Microsoft、流媒体、国内直连、全球直连、广告、下载等规则。
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)：Gemini / OpenAI / Copilot、YouTube、Netflix、Telegram、Apple、Microsoft、Game、GitHub 等应用规则；Claude 使用本仓库的高覆盖率专属规则。

## 关于旧模板中的节点 URL

旧版 `Merge.yaml` 曾把两个订阅 URL 直接写在公开仓库中。新版已移除这层硬编码，节点统一由 Sub-Store 的源订阅提供。若旧 URL 仍有效，建议视为已公开暴露并在对应服务商处轮换订阅链接。


## 节点选择与其他分组

- `♻️ 自动选择`：对除 Oracle 外的全部节点做 URL 测速，自动选择延迟较低的节点。
- `🌐 手动切换`：按国家进入手动节点列表；国家不再设置独立自动测速组，全局测速由 `♻️ 自动选择` 负责。
- `☁️ 甲骨文圣何塞`：精确筛选最终 YAML 中名为 `reality` 和 `hy2-hop` 的两个自建节点，不参加美国筛选、国家手动组和全局自动测速。
- `📺 YouTube`、`🎞️ 奈飞视频`：分别优先匹配，剩余流媒体进入 `🎬 流媒体`。
- `🌍 全球直连`：只对应 Sukka 的 `direct.txt` 清单，不代表所有海外网站都直连。
- 国内软件、国内域名、Apple / Microsoft CDN 等指定规则直接指向 `DIRECT`，不再额外设置「国内直连」策略组。
- `🛑 广告拦截`：默认 REJECT，可在组内切换 REJECT-DROP、DIRECT 或节点选择。
 
## 漏网之鱼的工作原理

`🐟 漏网之鱼` 不是额外的域名清单，而是最后一条 `MATCH` 规则。前面所有规则都没有命中的连接，才会落到这里。默认走 `🎯 节点选择`，也可以手动切成 DIRECT。它的作用是给未知流量一个明确、可控的兜底出口，避免流量因为没有匹配规则而行为不确定。


## OpenClash DNS / Fake-IP 设置

OpenClash 的运行模式、DNS 劫持、Dnsmasq 转发、DNS 分流、嗅探、QUIC 禁用和大陆 IP 绕过，均由 OpenClash 插件设置管理。使用本模板时，不需要在模板中再配置一遍，也不需要为了本模板额外开启 Fake-IP Filter、Nameserver-Policy 或自定义上游 DNS。

如果你已经在 OpenClash 中配置好 DNS 和嗅探，保持现有设置即可。模板没有 `dns` / `sniffer` / `tun` 区块，不会主动覆盖这些选项。


## Claude / Gemini 图标

Claude 和 Gemini 策略组图标使用 [Lobe Icons](https://github.com/lobehub/lobe-icons) 提供的彩色 SVG CDN：

- Claude：`https://unpkg.com/@lobehub/icons-static-svg@latest/icons/claude-color.svg`
- Gemini：`https://unpkg.com/@lobehub/icons-static-svg@latest/icons/gemini-color.svg`

使用 Lobe Icons 官方 [CDN Usage](https://github.com/lobehub/lobe-icons#-cdn-usage) 方式引用静态 SVG。
