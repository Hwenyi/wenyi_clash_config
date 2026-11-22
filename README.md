# wenyi_clash_config

个人 Mihomo / OpenClash 分流配置，使用 **Sub-Store 组合订阅 + 远程覆写模板** 生成最终配置。

## 文件

- \`mihomo-override.yaml\`：Sub-Store 使用的 Mihomo 远程覆写模板，不是独立节点订阅。
- \`rules/claude.list\`：Claude / Anthropic 高覆盖率规则，按“宁可错杀，不可放过”维护。

## Sub-Store 使用流程

1. 在 Sub-Store 中准备一个源订阅或组合订阅，确保其中包含你需要的机场节点，以及 Oracle 两个节点。
2. 确认 Sub-Store 输出的节点名称原样保留：
   - \`Oracle-vless-reality\`
   - \`Oracle-hy2-hop\`
3. 在 Sub-Store 的「文件」中创建 Mihomo 配置，选择该组合订阅作为配置来源，并使用远程覆写。
4. 远程覆写 URL：

   \`https://raw.githubusercontent.com/Hwenyi/wenyi_clash_config/main/mihomo-override.yaml\`

5. 使用 Sub-Store 生成的**最终配置文件 URL**导入 OpenClash。不要把本仓库的覆写文件 URL 直接当作完整订阅导入。

策略组使用 Mihomo 的 \`include-all: true\` 从 Sub-Store 最终配置中的节点集合读取节点，不在模板里再写一套机场订阅 URL。Oracle 组通过节点名称精确筛选，因此不会混入美国节点或全局自动测速组。

## Oracle 节点

\`☁️ 甲骨文圣何塞\` 只筛选以下两个精确名称：

- \`Oracle-vless-reality\`
- \`Oracle-hy2-hop\`

两个节点在同一个独立策略组中，可以在组内分别点选 VLESS + Reality 或 Hysteria 2。美国节点组不会匹配这两个名称。

## 国家节点筛选

自动测速和手动选择各有一组。美国除国家名、旗帜、US/USA 外，还识别洛杉矶、圣何塞、圣克拉拉、旧金山、芝加哥、达拉斯、西雅图、波特兰、纽约、阿什本、弗吉尼亚等常见机房城市。

筛选正则使用 Go / RE2 兼容语法，不使用 Lookahead / Lookbehind。

## AI 分流优先级

1. **Claude**：本仓库规则 + 官方 Anthropic 入站 IP + ASN + NTP。
2. **Gemini**：blackmatrix7 Gemini 规则，并补充 NotebookLM。
3. **AI**：OpenAI、ChatGPT、Copilot，以及 clash-rules-cn / Sukka 的综合 AI 规则。

Claude 规则有意把以下共享第三方服务也纳入 Claude：Sentry、Statsig、Datadog、Sift、Intercom、Fathom，以及包含 \`mcp\` 关键词的域名。这些服务也可能被其他应用使用，因此可能把其他应用的遥测或客服流量一起送入 Claude 策略组；这是本配置按用户要求采取的主动过匹配策略。

MCP 服务端域名由用户或第三方自行定义，无法仅靠域名规则识别所有不含 \`mcp\` 关键词的任意 MCP 主机；如果实际使用的 MCP 服务域名不含相关关键词，需要按实际域名补充到 \`rules/claude.list\`。

### Claude IP 范围

配置使用 Anthropic 官方公布的**入站**地址段：

- IPv4：\`160.79.104.0/23\`
- IPv4：\`153.61.224.0/22\`
- IPv6：\`2607:6bc0::/48\`

另参考 Net.Coffee 加入 \`AS399358\` 作为补充。没有把 Anthropic 官方文档中的出站网段当作客户端访问 Claude 的目标 IP：出站地址用于 Anthropic 向外部 MCP 服务发起请求，语义不同。

\`GEOSITE,category-ntp\` 也被送入 Claude 组，是为了覆盖 Net.Coffee 提到的 NTP / 时区一致性场景。它会影响设备的 NTP 流量；如果你不希望 NTP 使用 Claude 策略组，可以删除这条规则。

## OpenClash「绕过中国大陆」

本模板不再加载 \`china-ip\` / \`domestic-ip\` 规则集，也不写 \`GEOIP,CN,DIRECT\`。如果 OpenClash 已启用「绕过中国大陆」，保持开启即可；该功能在插件层绕过符合条件的大陆流量，重复添加大陆 IP 规则没有必要。

模板仍保留国内常见软件域名、国内直连域名、Apple 中国 CDN 等域名规则。局域网 / 私有地址规则也保留，因为它们不是大陆 IP 绕过的替代品。

## 规则来源

- [clash-rules-cn](https://github.com/mcxiaochenn/clash-rules-cn)：国内直连、常见软件、AI、GFWList、广告、私有域名等聚合规则。
- [Sukka Ruleset](https://ruleset.skk.moe/)：AI、Apple、Microsoft、流媒体、国内直连、全球直连、广告、下载等规则。
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)：Claude / Gemini / OpenAI / Copilot、YouTube、Netflix、Telegram、Apple、Microsoft、Game、GitHub 等应用规则。

## 关于旧模板中的节点 URL

旧版 \`Merge.yaml\` 曾把两个订阅 URL 直接写在公开仓库中。新版已移除这层硬编码，节点统一由 Sub-Store 的源订阅提供。若旧 URL 仍有效，建议视为已公开暴露并在对应服务商处轮换订阅链接。
