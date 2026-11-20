# Shadowrocket 纯净 AI 分流配置

一套专为 **AI 高风控服务（尤其是 Claude / Anthropic）** 优化的 Shadowrocket（小火箭）白名单分流配置，上游规则集基于 [`blackmatrix7/ios_rule_script`](https://github.com/blackmatrix7/ios_rule_script)。

## 核心设计

1. **Claude 独立「手动固定节点」策略组（`🤖 Claude 专线`）**
   - 使用 `select`（手动指定）而非 `url-test`（自动测速），彻底避免出口 IP 频繁跳变触发 Anthropic 风控。
   - 默认过滤掉香港/澳门及订阅信息条目，方便你直接选中唯一一个**干净的美国/日本/新加坡住宅或原生静态 IP 节点**。
   - 同时接管 `challenges.cloudflare.com`，保证 Cloudflare 人机验证与 `claude.ai` 使用完全一致的出口 IP。
2. **防泄露全局参数（`[General]`）**
   - `ipv6 = false` / `prefer-ipv6 = false`：关闭 IPv6，避免双栈环境或脏 IPv6 出口导致 AI 拒绝服务。
   - `block-quic = all-proxy`：屏蔽代理侧 QUIC（UDP 443），强制降级为更稳定的 TCP/HTTPS，减少特征暴露与断流。
   - `hijack-dns` + `private-ip-answer = true`：防止应用硬编码 DNS 导致解析泄露。
3. **主流 AI 与 Apple Intelligence 全覆盖（`🧠 通用 AI`）**
   - 涵盖 Claude / Anthropic、OpenAI (ChatGPT / Sora)、Google Gemini (AI Studio / NotebookLM)、Cursor、Copilot、Perplexity、Grok (x.ai) 以及 Apple Intelligence。
   - `AppleIntelligence.list` 优先级高于常规 `Apple.list`，避免被普通苹果直连规则截胡。
4. **白名单兜底（`🐟 漏网之鱼` 默认走 `🤖 Claude 专线`）**
   - 国内服务与局域网通过 `Lan.list` + `ChinaMax.list` + `GEOIP,CN` 走 `DIRECT`。
   - 高流量常见海外服务（YouTube、Netflix、Google、GitHub、Telegram）走日常/流媒体分组；其余未命中的海外未知域名通过 `FINAL,🐟 漏网之鱼` 默认走 **`🤖 Claude 专线`**（同一固定纯净 IP），彻底杜绝 AI 新增子域名或鉴权域名落入测速/香港节点带来的风控风险。

## 导入方法

1. 打开仓库中的 [`shadowrocket.conf`](./shadowrocket.conf)，点击 **Raw** 获取直链地址。
2. 打开 Shadowrocket -> 底部 **配置** -> 右上角 **`+`** -> 粘贴 Raw 链接并下载。
3. 点击下载好的配置，选择 **使用配置**。
4. 在 **首页 -> 全局路由** 中确认设置为 **配置**。

## 使用 Claude 前的必做检查

1. **固定节点**：在配置页或首页策略组中进入 **`🤖 Claude 专线`**，手动选中一个**非香港、IP 纯净且稳定**的节点（如你的美区住宅 IP 或专属静态节点），之后尽量不要随意切换。
2. **防泄露自检**：连上小火箭后，用手机浏览器访问：
   - `https://claude.ai/cdn-cgi/trace`
   - 确认返回结果中的 `ip=` 是你选定的固定节点 IP，且 `loc=` 为对应目标国家代码（例如 `US`、`JP`、`SG`），**绝对不能出现 `CN` 或 `HK`**。
3. **国内网络更新规则提示**：配置文件内引用了 `raw.githubusercontent.com` 的远程规则集，更新配置时建议先开启代理再点击更新。
