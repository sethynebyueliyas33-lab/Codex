# Skills.md

## 全 AGI 通用：Google Cloud 代理 / VPN 节点生成与订阅管理 Skill

> 面向 ChatGPT、Claude、Gemini、Codex 与其他 Agent 的通用工作说明书。
>
> 目标：让 AI 能够根据服务器环境选择协议、生成客户端配置、检查安全问题，并明确告诉用户哪些服务可能无法通过云服务器节点正常使用。

---

## 0. 使用原则

这份 Skill **只保存方法，不保存真实凭据**。

永远不要把以下内容提交到公开 GitHub：

- 真实服务器 IP / 域名
- UUID
- Token
- 密码
- 私钥
- 真实订阅地址
- Google Cloud 服务账号密钥
- 任何可以直接登录服务器或节点的认证信息

真实参数只能存放在：

- Google Secret Manager
- GitHub Actions Secrets
- 本地 `.env`
- 其他受保护的 Secret Store

公开仓库里只使用占位符。

---

# 1. AI 接到任务后应该先做什么

当用户让你“生成 VPN / 代理节点”时，不要直接吐一大段配置。

先判断：

1. 云服务器平台是什么？
2. 用户更看重速度、稳定性还是兼容性？
3. 客户端是什么？
4. 是否需要整机 TUN / VPN 模式？
5. 是否需要订阅链接？
6. 服务器 CPU / 内存是否很小？
7. 是否使用 Google Cloud Free Tier？
8. 目标网站是否会限制数据中心 IP 或 VPN？

然后再选择协议。

---

# 2. 协议怎么选

## 2.1 简单结论

如果不知道选哪个，可以按下面的顺序：

### 追求综合性能

**VLESS + REALITY + Vision / TCP**

推荐作为主要节点。

特点：

- 协议本身较轻量
- TCP 模式对小型 VPS 比较友好
- XTLS Vision 在合适环境下可以降低代理核心的 IO 开销
- 不需要为了普通用法再套一层 WebSocket
- 适合作为 Google Cloud 小型 VM 的主节点

推荐输出：

- VLESS URI
- Mihomo / Clash YAML
- sing-box JSON
- Shadowrocket 可导入链接

---

### 追求兼容性

**VLESS + WebSocket + TLS**

适合作为备用节点。

优点：

- 客户端兼容性高
- 可以与传统 HTTPS / WebSocket 架构结合
- 配置资料多

缺点：

- 相比直接 TCP 会多一层 WebSocket 封装
- 通常不是单纯追求速度时的首选
- 新部署不应把 WebSocket 当成唯一方案

---

### 网络丢包比较严重、移动网络波动大

**Hysteria 2**

特点：

- 基于 QUIC / UDP
- 对高延迟、丢包环境通常更有优势
- 很适合移动网络和质量不稳定的线路

但需要注意：

- QUIC 在用户态运行，比普通 TCP 更吃 CPU
- 极低配置 VPS 可能先撞上 CPU 瓶颈
- UDP 被限制的网络中可能无法工作
- 不建议把它作为 `e2-micro` 的唯一节点

推荐定位：

**高速备用节点 / 移动网络节点**。

---

### 只想要最简单的整机 VPN

**WireGuard**

特点：

- 协议简单
- 开销低
- 非常适合系统级全局 VPN
- 通常能提供很好的吞吐量和延迟

缺点：

- 它不是 VLESS / Clash 生态里的“订阅节点”思路
- 多客户端订阅管理方式与代理协议不同

推荐定位：

**纯 VPN / 全设备隧道优先选择。**

---

### SOCKS5

SOCKS5 只是代理接口，不应被当成现代加密 VPN 协议。

默认规则：

- 不允许裸 SOCKS5 直接暴露公网
- 不把 SOCKS5 用户名密码当成加密
- 只作为本机、私网、SSH 隧道或其他受保护环境中的备用入口

---

## 2.2 实用排名

| 需求 | 推荐协议 | 结论 |
| --- | --- | --- |
| 综合主节点 | VLESS + REALITY + Vision | ⭐⭐⭐⭐⭐ |
| 低配置 Google Cloud VM | VLESS + REALITY / TCP | ⭐⭐⭐⭐⭐ |
| 客户端兼容性 | VLESS + WS + TLS | ⭐⭐⭐⭐ |
| 高丢包 / 移动网络 | Hysteria 2 | ⭐⭐⭐⭐⭐ |
| 整机 VPN | WireGuard | ⭐⭐⭐⭐⭐ |
| 临时本地代理 | SOCKS5 | ⭐⭐ |

> “最快”没有绝对答案。最终速度取决于服务器位置、网络质量、CPU、拥塞、客户端和网站本身。

---

# 3. Google Cloud 免费服务器运行方案

## 3.1 Free Tier 默认选择

如果用户明确要求“Google Cloud 免费服务器”，优先使用 Compute Engine Free Tier 支持的 `e2-micro`。

当前 Free Tier 的核心限制：

- 每月最多相当于 1 台非抢占式 `e2-micro` 的运行时长
- Free Tier 仅适用于以下美国区域：
  - `us-west1`（Oregon）
  - `us-central1`（Iowa）
  - `us-east1`（South Carolina）
- 每月 30 GB 标准 Persistent Disk
- 每月仅 1 GB 免费出站流量（北美到部分目的地区域）

**非常重要：Free Tier 不等于无限流量。**

VPN / 代理最容易产生费用的地方不是 VM 本身，而是网络出站流量。

如果用户准备：

- 看高清视频
- 下载大型文件
- 长时间测速
- 多设备共用

必须先提醒：**1 GB 免费流量很快就会用完，超出后可能计费。**

---

## 3.2 地区选择

默认优先：

```text
us-west1
```

原因：如果用户在东亚，通常美国西海岸比美国中部、东海岸延迟更低。

但必须实际测试，不能把地区名称直接当成速度结论。

注意：

- Tokyo `asia-northeast1`
- Osaka `asia-northeast2`

虽然有 Compute Engine，但**不属于当前 `e2-micro` Free Tier 的免费区域**。

如果用户选择东京或大阪，要明确告诉用户可能产生正常 Compute Engine 费用。

---

# 4. 默认部署结构

推荐主节点：

```text
客户端
  ↓
VLESS + REALITY / TCP
  ↓
Google Cloud e2-micro
  ↓
互联网
```

备用节点：

```text
客户端
  ↓
VLESS + WebSocket + TLS
  ↓
Google Cloud
  ↓
互联网
```

网络条件较差时可额外提供：

```text
客户端
  ↓
Hysteria 2 / QUIC
  ↓
Google Cloud
  ↓
互联网
```

如果只是个人整机 VPN：

```text
设备
  ↓
WireGuard
  ↓
Google Cloud
  ↓
互联网
```

---

# 5. 节点输出规则

Agent 生成节点后，优先同时输出以下格式：

1. 人类可读摘要
2. VLESS / 对应协议 URI
3. Mihomo / Clash YAML
4. sing-box JSON
5. Shadowrocket 兼容导入格式
6. 参数检查表

节点名称建议：

```text
GCP-US-West-VLESS
GCP-US-West-WS
GCP-US-West-HY2
GCP-US-West-WG
```

不要把用户名、邮箱、真实姓名写进节点名称。

---

# 6. 访问能力与限制

## 6.1 必须先说明

Google Cloud 的公网 IP 属于**数据中心 IP**。

网站可以识别：

- IP 所属云服务商
- ASN
- 国家 / 地区
- 是否存在异常登录行为
- 是否疑似 VPN / Proxy

因此：

**节点能连通互联网 ≠ 所有网站都愿意接受这个节点。**

本 Skill 不承诺“流媒体解锁”“银行风控绕过”或绕过任何第三方网站的访问限制。

如果网站明确禁止代理 / VPN，Agent 不应继续尝试规避其反代理策略。

---

## 6.2 通常可以正常访问

以下类型网站通常只要求普通 HTTPS 连接，因此一般可以正常使用：

- GitHub
- Wikipedia
- Google Search / Google Docs 公共页面
- 普通技术文档网站
- 大部分新闻与博客网站
- 常规静态网站
- 普通软件下载站

但仍可能遇到验证码、登录验证或 IP 风控。

---

## 6.3 可以访问，但可能触发额外验证

以下服务经常会根据数据中心 IP、登录地点和账户历史触发额外风控：

- X
- Reddit
- Discord
- YouTube
- Google 账户登录
- ChatGPT / Claude / Gemini 等在线 AI 服务
- 电商网站

可能出现：

- CAPTCHA
- 邮箱验证
- 手机验证
- 异常登录提醒
- 临时限速
- 地区可用性提示

是否能正常使用由网站自身规则决定。

---

## 6.4 不应承诺可用 / 经常受限制

### Netflix

不要把 Google Cloud 数据中心 IP 当成 Netflix 解锁节点。

Netflix 官方明确说明：

- VPN / Proxy 可能触发代理检测错误
- 广告套餐不支持 VPN
- Netflix 直播内容不支持 VPN
- 使用 VPN 时可能只显示 Netflix 拥有全球授权的部分内容

因此默认标记：

```text
Netflix：⚠️ 不保证
```

### 《龙珠超》

Netflix 日本当前页面明确显示《龙珠超》在当地不可观看。

因此本 Skill 必须直接标记：

```text
Netflix《龙珠超》：❌ 不作为可访问内容承诺
```

不要告诉用户“换一个 Google Cloud 节点就一定能看”。

内容版权、地区授权和 Netflix 的 VPN 检测是三个不同问题。

---

### 其他流媒体

默认全部视为“可能受限”，包括但不限于：

- Disney+
- Hulu
- Prime Video
- Max
- 其他存在地区版权库的流媒体服务

Agent 可以做普通连通性测试，但不得保证某个机房 IP 一定能解锁某个国家的片库。

---

## 6.5 高风控网站

以下网站即使技术上能连接，也可能拒绝数据中心 IP：

- 银行
- 支付平台
- 加密货币交易平台
- 政府身份服务
- 票务网站
- 游戏账号 / 反作弊服务

对于这些服务，优先建议使用用户正常、可信的本地网络，不要把云服务器代理当成身份验证网络。

---

# 7. 访问矩阵输出模板

每次生成节点后，Agent 都应该附上：

| 服务 | 状态 | 说明 |
| --- | --- | --- |
| GitHub | ✅ 通常可用 | 普通 HTTPS |
| Wikipedia | ✅ 通常可用 | 普通 HTTPS |
| Google Search | ✅ 通常可用 | 可能 CAPTCHA |
| YouTube | ⚠️ 条件可用 | 地区 / 风控 / 速度影响 |
| X | ⚠️ 条件可用 | 可能触发登录验证 |
| ChatGPT | ⚠️ 条件可用 | 受账号和地区政策影响 |
| Netflix | ⚠️ 不保证 | 可能检测 VPN / 数据中心 IP |
| Netflix《龙珠超》 | ❌ 不承诺 | 当前日本 Netflix 页面显示不可用；且 VPN 还可能进一步限制片库 |
| Disney+ / Hulu / Prime Video / Max | ⚠️ 不保证 | 流媒体地区与代理检测 |
| 银行 / 支付 | ⚠️ 不推荐 | 容易触发风控 |

Agent 不得把“网页能打开”误写成“流媒体内容已解锁”。

---

# 8. 安全配置规则

## 8.1 VLESS

- 必须配合安全传输层使用
- UUID 必须随机生成
- 不复用公开示例 UUID
- 不同用户建议使用不同凭据
- 只开放必要端口

## 8.2 TLS

- 优先 TLS 1.3
- 使用有效证书
- 配置证书自动续期
- 不关闭证书校验来“解决问题”

## 8.3 Google Cloud Firewall

默认拒绝不需要的入站。

只开放实际使用的端口。

SSH 管理入口应尽量限制来源，并优先使用 Google Cloud 的安全登录机制、密钥或 IAP。

不要将管理面板暴露到 `0.0.0.0/0`。

---

# 9. 凭据占位符

统一使用：

| 占位符 | 含义 |
| --- | --- |
| `GCP_PROJECT_ID` | Google Cloud 项目 |
| `GCP_REGION` | VM 区域 |
| `GCP_ZONE` | VM Zone |
| `SERVER_HOST` | 服务器地址 |
| `SERVER_PORT` | 节点端口 |
| `CLIENT_UUID` | VLESS UUID |
| `REALITY_PUBLIC_KEY` | REALITY 公钥 |
| `REALITY_PRIVATE_KEY` | REALITY 私钥（不得公开） |
| `REALITY_SHORT_ID` | REALITY Short ID |
| `WS_PATH` | WebSocket 路径 |
| `HY2_PASSWORD` | Hysteria 2 密码 |
| `WG_PRIVATE_KEY` | WireGuard 私钥（不得公开） |
| `SUBSCRIPTION_TOKEN` | 订阅访问 Token |

---

# 10. 测试流程

节点生成后依次测试：

1. DNS
2. TCP / UDP 连通性
3. TLS / REALITY 握手
4. 代理认证
5. 普通 HTTPS 网站
6. 延迟
7. 下载吞吐
8. DNS 泄漏
9. IPv6 路由
10. 网站访问矩阵

测速时避免连续跑大流量测试，因为 Google Cloud Free Tier 的免费出站很少。

---

# 11. Agent 最终回答模板

每次部署完成后，最终回答必须包含：

```text
✅ 服务器：Google Cloud / REGION
✅ VM：e2-micro
✅ 主节点：PROTOCOL
✅ 备用节点：PROTOCOL
✅ 客户端：CLIENT
✅ 订阅格式：FORMAT

速度建议：...
兼容性建议：...
预计费用：...
免费额度风险：...

访问情况：
GitHub：✅
Google：✅ / ⚠️
X：⚠️
YouTube：⚠️
Netflix：⚠️ 不保证
Netflix《龙珠超》：❌ 不承诺
银行 / 支付：⚠️ 不建议走云代理
```

如果没有实际测试过，就必须写“预计 / 未验证”，不能伪造测速或解锁结果。

---

# 12. Google Cloud 成本保护

部署前必须：

- 开启 Billing Budget
- 设置费用提醒
- 检查 Network Egress
- 不假设免费 VM = 免费流量
- 不通过创建多个项目或账号规避免费额度限制
- 遵守 Google Cloud AUP 与服务条款

如果用户只想体验这套 Skill，推荐小流量测试，不建议使用 Free Tier 节点长期观看高清视频。

---

# 13. 参考资料

Google Cloud Free Tier：
https://cloud.google.com/free

Google Cloud Free Program 文档：
https://docs.cloud.google.com/free/docs/free-cloud-features

Google Cloud Acceptable Use Policy：
https://cloud.google.com/terms/aup

Xray / Project X：
https://xtls.github.io/

VLESS：
https://xtls.github.io/en/config/outbounds/vless.html

Xray Transport：
https://xtls.github.io/en/config/transport.html

Hysteria 2：
https://v2.hysteria.network/

sing-box：
https://sing-box.sagernet.org/

WireGuard：
https://www.wireguard.com/

Netflix VPN 帮助：
https://help.netflix.com/en/node/114701

---

# 14. 最终原则

> **Google Cloud 提供服务器，协议负责传输，Skill 负责把经验留下来，Agent 负责执行。**

> **不要把“能连接”写成“什么都能看”；不要把“免费 VM”写成“无限免费流量”。**

> **公开仓库只存方法，真实节点凭据永远留在私有环境。**
