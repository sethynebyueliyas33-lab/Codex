---
name: aws-ec2-hysteria
description: 使用 AWS Agent Toolkit for AWS 检查 AWS 项目与区域、规划 EC2 云服务器，并在获得授权后部署和验收 Hysteria 2。适用于 AWS Agent Toolkit、AWS 新版 Experience、EC2、Ubuntu、Hysteria 2、UDP 443 或 Clash 订阅配置相关任务。
compatibility: 需要可访问 AWS 官方文档和 GitHub 的网络环境；执行 AWS 资源操作必须使用用户授权的 AWS CLI profile，并在执行前确认目标项目、区域与费用。
---

# AWS EC2 + Hysteria 2 实战 Skill

本 Skill 记录 AWS Agent Toolkit 与 EC2 / Hysteria 2 的可复用流程。它是方法说明，不包含用户真实服务器参数或凭据。

## 1. 已有实践记录（截至 2026-11-25）

根据用户提供的操作截图，已观察到以下一次部署结果：

- AWS Experience：New AWS Experience。
- 项目资源区域：`ap-southeast-2`（Asia Pacific / Sydney）。
- 由于该项目页面拒绝切换到新加坡或东京，本次选择继续在悉尼部署。**这是该项目当时显示的区域限制，不代表所有 AWS 账户都只能使用悉尼区域。**
- EC2 实例规格：`t3.micro`。
- 操作系统：Ubuntu 24.04。
- EBS：8 GiB gp3。
- 安全组截图显示 TCP 22 与 UDP 443 已开放；SSH 来源范围需要在 AWS 控制台进一步核实，建议仅允许可信管理 IP。
- Hysteria 2：截图显示版本 `v2.14.0`，服务状态为 `active`，UDP 443 正在监听。
- TLS：截图记录使用自签证书与 `pinSHA256` 校验；客户端配置使用 pin 校验，`insecure` 保持关闭；SNI 显示为 `www.bing.com`。
- Agent 表示已生成二维码和 Clash 订阅配置，但截图没有证明客户端已经完成端到端连接、DNS、网页访问或长期稳定性测试。

以上是一次实践记录，不是所有 AWS 账户或网络环境的通用保证。每次新部署都必须重新核实区域、权限、版本、价格和客户端兼容性。

## 2. 使用 AWS Agent Toolkit 前的检查

官方仓库与安装说明：

- Repository: https://github.com/aws/agent-toolkit-for-aws
- Setup instructions: https://raw.githubusercontent.com/aws/agent-toolkit-for-aws/refs/heads/main/setup-instructions/setup.md
- AWS quick start: https://docs.aws.amazon.com/agent-toolkit/latest/userguide/quick-start.html

按官方 setup 文档执行，不要跳过前置条件：

1. 自动检测操作系统（Linux / macOS / Windows）。
2. 检查 AWS CLI v2 是否已安装。
3. 检查 `uv` 是否可用；按官方文档安装缺失组件。
4. 检查能否访问 `https://awscli.amazonaws.com`。
5. 询问并确认 AWS CLI profile 名称。
6. 确认用户属于 New AWS Experience 还是 Advanced AWS Experience。
7. 确认项目的默认区域；New AWS Experience 应使用项目创建时绑定的区域。
8. 使用官方 `aws login` / `aws signup` 浏览器流程完成身份验证，之后用 `aws sts get-caller-identity --profile <profile>` 核实身份。
9. 按官方文档执行 `aws configure agent-toolkit`，让用户完成交互式向导。

安全边界：

- 绝不向用户索要或写入 AWS Secret Access Key、密码或会话令牌。
- 不要把凭据写进仓库、终端截图、日志或聊天内容。
- 官方 setup 文档说明：登录凭据有效期为 12 小时，可在 90 天内续期而无需重新进行浏览器认证；实际状态以当前 CLI 提示为准。
- Agent Toolkit 服务端当前要求使用 `us-east-1`；按官方 setup 指令运行 toolkit 安装 / skill 查询命令时，不要把用户 EC2 项目的区域替换进去。**Toolkit 服务区域与 EC2 资源区域是两个不同概念。**

## 3. 区域与项目边界

- 对 New AWS Experience，先从项目设置确认项目区域，再把 EC2 资源部署到该项目允许的区域。
- 若项目页面提示无法切换区域，不要反复尝试创建其他区域资源或声称可以绕过限制；记录页面提示，并在当前项目允许的区域继续，或让用户联系 AWS 支持确认项目资格。
- 本次用户实践的资源区域为 `ap-southeast-2`（悉尼）。
- `us-east-1` 是 Agent Toolkit 服务 / 指令要求的区域，不等于本次 EC2 服务器部署区域。
- 不得将一次账户的项目区域限制写成 AWS 所有账户的普遍规则。

## 4. EC2 资源规划

本次截图中的实例规格可作为历史案例，不应无条件复制：

| 项目 | 本次记录 |
| --- | --- |
| Region | `ap-southeast-2` |
| Instance type | `t3.micro` |
| OS | Ubuntu 24.04 |
| EBS volume | 8 GiB gp3 |
| Inbound | TCP 22、UDP 443 |
| Service | Hysteria 2 |

部署前必须核实：

- 项目是否有权限创建目标实例与网络资源。
- 当前账户的 Free Tier / credits 是否适用，实例类型、操作系统镜像、磁盘、运行时长是否满足资格。
- 公网 IPv4、EBS、出站流量、快照、日志及其他配套资源是否产生费用。
- 预算告警是否已配置。预算告警通常只是通知，不等同于硬性消费上限或自动关机。
- 使用 AWS Pricing Calculator 或 AWS 官方定价页估算目标区域的费用；不要把 `t3.micro` 或 gp3 直接描述为永久免费。

官方参考：

- EC2 Free Tier usage: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-free-tier-usage.html
- EC2 On-Demand pricing: https://aws.amazon.com/ec2/pricing/on-demand/
- VPC pricing: https://aws.amazon.com/vpc/pricing/
- AWS Pricing Calculator: https://calculator.aws/

## 5. 网络与安全组

最小化开放端口：

- TCP 22：仅用于 SSH 管理。来源 CIDR 应限制为用户当前可信公网 IP 或专用管理网络；不要默认开放到 `0.0.0.0/0`。
- UDP 443：仅当 Hysteria 2 服务确实监听该端口时开放。
- 不要为了“测试方便”开放所有 TCP / UDP 端口。
- 同时检查实例内 Ubuntu 防火墙、AWS Security Group 与服务实际监听地址；三者必须一致。
- 不公开实例 ID、公网 IP、二维码、有效 Clash 订阅 URL、密码或证书私钥。

AWS Security Group 官方说明：
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html

## 6. Hysteria 2 部署与配置

本次用户截图记录：

- Hysteria 2 `v2.14.0`。
- 服务状态 `active`。
- UDP 443 正在监听。
- 自签证书与 `pinSHA256` 校验。
- 客户端关闭 `insecure`。
- SNI 记录为 `www.bing.com`。
- 已生成二维码与 Clash 订阅配置。

通用要求：

1. 部署前检查 Hysteria 官方最新 Release 与安全公告；版本号不可永久固定。
2. 只在用户授权的 EC2 实例上安装服务。
3. 生成强随机密码与证书材料；私钥和密码不得进入 Git。
4. 如果采用自签证书，必须正确核对证书指纹 / pin，并在客户端保持证书校验开启；不得把 `insecure: true` 当作常规解决方案。
5. SNI、证书指纹、服务器证书与客户端配置必须相互匹配；不要仅复制示例值。
6. 验证 systemd 服务状态、UDP 端口监听、实例安全组与主机防火墙。
7. 生成 Clash 配置时只使用占位符写入文档，不把真实订阅地址或二维码提交到仓库。
8. Hysteria 服务处于 active 且 UDP 端口监听，只能证明服务进程和监听状态正常；不能单独证明客户端握手、DNS、网页访问或速度测试成功。

官方文档与 Release：

- Hysteria 2 documentation: https://v2.hysteria.network/
- Hysteria releases: https://github.com/HyNetworks/hysteria/releases

## 7. 验收清单

每次部署按以下项目逐项记录结果：

- [ ] AWS CLI profile 指向预期 AWS 身份 / project。
- [ ] EC2 实例区域与项目授权区域一致。
- [ ] 实例规格、系统镜像、磁盘与预计费用已核实。
- [ ] SSH 仅允许可信来源访问。
- [ ] UDP 443 在安全组与主机防火墙中按需放行。
- [ ] Hysteria 服务处于 active。
- [ ] UDP 443 正在监听。
- [ ] 客户端 pinSHA256 与服务端证书匹配，未关闭证书校验。
- [ ] Clash 配置已导入并通过客户端连接测试。
- [ ] DNS、IPv4 / IPv6、目标网站与实际流量已测试。
- [ ] AWS Billing / Cost Explorer 中已核实实际费用。

没有证据的项目必须写“未测试”或“待核实”，不得把计划、命令执行成功或服务监听状态写成完整端到端验收。

## 8. 公开仓库规则

只提交可复用方法、占位符、官方文档链接和去敏后的测试结果。

禁止提交：

- AWS Account ID、Access Key、Secret Key、Session Token
- EC2 公网 IP、实例 ID、真实主机名
- Hysteria 密码、证书私钥、真实 pin / 订阅 Token
- 可直接导入使用的二维码、Clash 订阅链接或完整节点 URI
- 含有上述信息的日志、终端截图或配置文件

推荐使用占位符：

```text
AWS_PROFILE=<profile_name>
AWS_REGION=ap-southeast-2
EC2_INSTANCE_ID=<redacted>
SERVER_HOST=<redacted>
HYSTERIA_PORT=443
HYSTERIA_PASSWORD=<stored-in-secret-manager>
TLS_CERT=<certificate-path>
TLS_KEY=<private-key-path>
TLS_PIN_SHA256=<redacted>
```

## 9. 维护规则

- 先读官方 Agent Toolkit setup / troubleshooting 文档、AWS EC2 文档与 Hysteria 官方 Release。
- 对区域资格、费用、Free Tier、版本和安全变更记录核查日期与官方来源。
- 只有存在实质变化或新增已验证实践时才提交更新。
- 不覆盖用户已有内容；更新文件前先获取最新 SHA。
- 不保存真实账号、服务器或订阅凭据。
