---
name: gcp-proxy
description: 规划、检查与生成 Google Cloud 上的个人 VPN / 代理节点及订阅配置；比较 WireGuard、VLESS / Xray、Hysteria 2，核对免费额度、访问限制和凭据安全。适用于用户提出 Google Cloud、代理协议、节点订阅、Mihomo、sing-box 或 Shadowrocket 配置相关任务时。
compatibility: 需要可访问官方文档的网络环境；执行云部署需要用户授权的 Google Cloud 权限。
---

# Google Cloud 节点与订阅配置 Skill

这是跨 Agent 可复用的任务规范。完整的中文背景、协议说明和访问矩阵见仓库根目录 [Skills.md](../../../Skills.md)。

## 工作流程

1. **先确认目标**：个人 VPN 还是应用代理？客户端类型？预算、地区、带宽、是否需要订阅文件？优先复用用户已提供的条件，缺少必要条件时再询问。
2. **核实最新规则**：先查看 Google Cloud 免费层、Compute Engine 定价、网络出站计费、客户端和协议的官方文档。对实时价格、资格、可访问性引用来源和核查日期。
3. **提出方案**：
   - WireGuard：适用于整机 VPN / 简洁加密隧道。
   - VLESS + REALITY / Vision：适用于 Xray 生态中的个人代理；须确认客户端支持。
   - VLESS + WebSocket + TLS：适用于需要 WebSocket 和传统 TLS 架构的情境。
   - Hysteria 2：适用于某些高延迟或丢包线路；先确认 UDP 网络可用、CPU 与客户端兼容。
   - SOCKS5：本身不是加密 VPN，不要裸露到公网。
   **不对协议做无条件速度排名**；实际吞吐取决于 CPU、链路、地理位置、拥塞和配置。
4. **先算费用再部署**：确认 Free Tier 区域与实例规格、磁盘、外部 IPv4、出站计费和超额规则。注意免费层对中国和澳大利亚目的地的免费出站限制；普通 Billing Budget 通知并不自动封顶费用。
5. **输出安全配置**：按用户所需生成 VLESS URI、Mihomo YAML、sing-box JSON、WireGuard 配置说明等；只在用户授权且工具可用时实施部署。使用 `SERVER_HOST`、`CLIENT_UUID`、`SUBSCRIPTION_TOKEN` 等占位符，不要编造真实连接成功状态。
6. **安全和访问限制**：不要把明文密码、UUID、私钥、有效订阅链接、云平台服务账号密钥提交公开 GitHub。云 IP 不保证 Netflix、Disney+、Prime Video 等流媒体片库解锁；对银行、支付等高风控服务提出限制和风险。不要把网站首页可访问当成具体内容可播放。
7. **验收**：逐项说明是否实测过连接、握手、DNS / IPv6、性能、客户端导入、目标网站和实际账单。如果未验证，明确写“未测试”。

## 最终输出顺序

- 推荐方案与理由
- 成本说明（地区、免费范围、收费风险）
- 可复制但无真实密钥的配置 / 命令
- 测试与错误排查方法
- 网站访问限制与不确定性
- 官方资料及核查日期

## 持续维护

定期审阅官方变更时，优先更新根目录 `Skills.md`，并同步修订本 Skill 中实质过期的工作规则。**无实质变化不提交**。变更前先获取最新文件 SHA；不要覆盖用户新增内容。准确引用官方来源；有争议时保留旧版本并提出核查建议。

官方资料：

- Google Cloud Free Tier：https://docs.cloud.google.com/free/docs/free-cloud-features
- Google Cloud 预算：https://docs.cloud.google.com/billing/docs/how-to/budgets
- Agent Skills 规范：https://agentskills.io/specification
- Xray：https://xtls.github.io/
- Hysteria 2：https://v2.hysteria.network/
- sing-box：https://sing-box.sagernet.org/
- WireGuard：https://www.wireguard.com/
- Netflix VPN：https://help.netflix.com/en/node/114701
