---
name: proxy-rules
description: Build and apply software-agnostic direct-routing policies for scholarly platforms and domestic university sites across Clash/Mihomo, PAC/system proxy, and other domain-based proxy clients, while keeping Google Scholar on a valid proxy path.
metadata:
  short-description: 跨代理软件持久化配置学术与高校网站直连
---

# 学术与高校网站直连（跨代理软件）

为用户维护一份与代理软件无关的域名分流策略，再按当前使用的 Clash/Mihomo、系统代理/PAC 或其他代理客户端生成对应规则。默认目标是让学术数据库、出版社和国内高校网站走直连，同时让 `scholar.google.com` 保持代理。

## 适用范围

- 用户要求把科研网站、文献数据库、出版社或国内高校官网/研究生系统改为直连、核验路由，或修复代理软件更新后失效的规则。
- 默认域名清单见 [references/default-domains.yaml](references/default-domains.yaml)。用户点名的域名优先于默认清单，并应写入通用清单，而不是只写进某个代理客户端的临时配置。
- 默认高校 IP 全文入口包括 IEEE Xplore（`ieeexplore.ieee.org`）、Elsevier/ScienceDirect（`elsevier.com`、`sciencedirect.com`）、ACM Digital Library（`dl.acm.org`）、SpringerLink（`link.springer.com`）、Wiley Online Library（`onlinelibrary.wiley.com`）、ACS/APS/AIP/RSC（`pubs.acs.org`、`journals.aps.org`、`pubs.aip.org`、`pubs.rsc.org`）、IOP（`iopscience.iop.org`）、Oxford Academic（`academic.oup.com`）、SAGE（`journals.sagepub.com`）、ASME、Optica、AIAA、LWW、JAMA、Lancet、Cochrane、Project Euclid、Project MUSE、World Scientific、IOS Press 和 IGI Global；完整域名以该参考文件为准。
- 国内高校域名按根域匹配：例如 `hit.edu.cn` 覆盖哈工大官网、`hitgs.hit.edu.cn` 研究生院和 `yzb.hit.edu.cn` 研招网；`hrbeu.edu.cn` 覆盖哈工程官网、`yjsy.hrbeu.edu.cn` 研究生院和 `yjs.hrbeu.cn` 研究生系统。新增学校前先核对官方根域。
- 按用户当前偏好，默认启用 `edu.cn` 广泛后缀，覆盖国内高校官网、研究生院、研招网和校内服务子域名；如果用户后续只要求个别学校，必须在写入前切换为具体根域清单。
- 不处理硬件驱动、网卡驱动或系统网络驱动；不修改与分流无关的代理节点、订阅地址和凭据。

## 通用策略模型

先建立一个中立的策略模型，再选择适配器：

- `direct_domains`：学术数据库、出版社、国内高校网站的域名后缀，动作是 `DIRECT`。
- `proxy_domains`：必须走代理的域名，默认只有 `scholar.google.com`。不得把整个 `google.com` 放入直连清单。
- `system_proxy_bypass_domains`：必须完全绕过系统代理/PAC 的域名，典型是依赖校园 IP 认证的 Web of Science/Clarivate；这与代理软件内部的 `DIRECT` 是两个层级。

`direct_domains` 已包含常见出版社全文入口（IEEE Xplore、ScienceDirect、SpringerLink、Wiley Online Library、ACM Digital Library、ACS/APS/AIP/RSC、IOP、Oxford Academic、SAGE 等）。这些域名只表示应走直连；若某站点的高校 IP 认证仍被系统代理或 PAC 影响，适配层还必须按用户确认将其加入完全绕过清单。

根域规则优先使用后缀匹配，避免只覆盖首页而漏掉登录、静态资源和 PDF 下载子域名。不要用泛化的 `.cn`、单个 IP 或整个 `google.com` 替代明确的域名清单。

## 代理软件适配

### Clash / Mihomo

- 使用 `DOMAIN-SUFFIX,<domain>,DIRECT`，并放在现有规则之前。
- `scholar.google.com` 必须先于直连规则，并指向当前配置中真实存在且不含 `DIRECT` 的代理组；不要假设代理组永远叫 `🔎 Google`、`自动选择` 或其他固定名称。
- 单个订阅可用当前规则扩展文件的 `prepend` 段；跨订阅、跨 UID 应优先使用全局 `profiles\\Script.js`，在 `main(config, profileName)` 中动态注入规则，并先去除脚本此前注入的同域名规则。
- 不要把生成的 `clash-verge.yaml` 当作持久化源，也不要整体覆盖订阅的 `rules` 列表。订阅更新后必须重新检查全局脚本是否仍被加载。

### Windows 系统代理和 PAC

- 如果用户要求网站完全不进入代理软件，在 Clash Verge 中检查 `system_proxy_bypass`，在 Windows `ProxyOverride` 中加入对应的主域和通配子域，例如：

  `webofscience.com;*.webofscience.com;clarivate.com;*.clarivate.com`

- 使用 PAC 的客户端应在 `FindProxyForURL(url, host)` 中按域名后缀返回 `DIRECT`；Google Scholar 返回代理。不要只修改 Mihomo 规则后声称系统代理已绕过。
- 不要擅自关闭或开启系统代理；只在用户授权的范围内维护绕过清单，并保留现有的本地网络绕过项。

### 其他代理软件

- 先识别软件、版本、运行模式和持久化规则入口，再把同一份通用清单转换为该软件的域名后缀规则、绕过列表或 PAC 条目。
- 不猜测配置字段名，也不把 Clash 专用语法直接写入其他软件。若软件不支持持久化域名规则，保留通用清单并明确需要用户在软件界面导入/粘贴。
- 用户更换代理软件时，复用 `direct_domains`、`proxy_domains` 和 `system_proxy_bypass_domains`，只重生成适配层；不要重新手工维护一套互相漂移的域名列表。

## 执行流程

### 1. 发现当前环境

检查正在运行的代理客户端、配置根目录和当前活动配置。Windows 上常见 Clash Verge Rev 数据目录是：

`%APPDATA%\\io.github.clash-verge-rev.clash-verge-rev`

读取 `profiles.yaml` 或软件等价的活动配置，不要凭文件名猜测当前订阅。记录当前规则入口、代理组名称、系统代理/PAC 状态和是否有服务模式。

### 2. 读取和扩展通用清单

需要广泛配置时读取 [references/default-domains.yaml](references/default-domains.yaml)。用户点名的高校或平台追加到清单；同一根域只保留一条后缀规则。默认清单已按用户偏好启用 `edu.cn`；如果用户明确限定为某几所高校，则先移除广泛后缀，改用具体根域。

启用 `edu.cn` 会覆盖所有中国高校子域名，必须在变更摘要中明确说明范围。哈工大和哈工程的具体根域始终可单独启用。

### 3. 生成适配层并写入

先备份将要修改的规则文件或应用设置，再按软件适配层写入。备份和日志中不得输出订阅 URL、token、密码或节点凭据。只修改用户指定的分流和必要的系统代理绕过项。

### 4. 重载和验证

- Clash/Mihomo 使用当前内核的配置校验命令，例如：

  `verge-mihomo.exe -t -d <clash-data-dir> -f <generated-config>`

- 服务模式下通过软件界面或服务控制通道重载，不要强制结束受保护的内核服务进程。
- 检查生成配置中 Google Scholar 规则位于直连规则之前、目标代理组存在，学术/高校域名的直连规则位于 `rules:` 前部。
- 在运行日志中核对 `using DIRECT` 或该软件等价的直连标记；Google Scholar 应显示代理出口。
- 用 `curl --proxy http://127.0.0.1:<port>` 测试时，只能验证代理软件内部的规则。显式指定代理会绕过 Windows `ProxyOverride`，不能用来证明系统代理绕过成功。
- 连接成功与机构授权分开判断：校园数据库可能要求出口 IP 在学校授权段内，直连本身不会授予订阅权限。

## 安全和变更边界

- 规则修改必须在用户明确要求后执行；仅诊断时不要擅自改配置。
- 不修改硬件驱动、网卡设置、系统安全策略或无关应用配置。
- 不把所有 `google.com`、所有 `.cn` 或未知镜像/下载站默认设为直连。
- 若订阅更新、软件更换或代理组变更导致验证失败，先停在诊断结果，不把规则指向猜测的出口组名。
- 完成后报告通用清单位置、适配层位置、验证结果和仍需用户在新代理软件中导入的部分。
