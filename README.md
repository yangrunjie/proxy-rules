# 代理规则（proxy-rules）

这是一个 Codex Skill，用于维护与代理软件无关的域名分流策略，并按 Clash/Mihomo、Windows 系统代理/PAC 或其他域名代理客户端生成适配规则。

## 主要用途

- 让学术数据库、文献检索站点、出版社和全文平台优先走直连，以便使用高校公网 IP 和机构订阅下载论文。
- 让国内高校官网、研究生院、研招网及校内服务域名走直连。
- 保持 `scholar.google.com` 走代理；不会把整个 `google.com` 加入直连。
- 在需要时区分代理软件内部 `DIRECT` 与 Windows 系统代理/PAC 的完全绕过。

## 默认覆盖范围

默认清单包含 Web of Science/Clarivate、Scopus、IEEE Xplore、ScienceDirect/Elsevier、ACM Digital Library、SpringerLink、Wiley、ACS、APS、AIP、RSC、IOP、Oxford Academic、SAGE、ASME、Optica、AIAA、JAMA、Lancet、Cochrane、Project Euclid、Project MUSE、World Scientific、IOS Press、知网、万方，以及常用开放学术平台。

国内高校默认启用 `edu.cn` 广泛后缀，并单独记录哈工大 `hit.edu.cn` 和哈工程 `hrbeu.edu.cn` 的官方根域。完整域名以 [references/default-domains.yaml](references/default-domains.yaml) 为准。

## 关键注意事项

1. 直连只表示流量不经远端代理节点；如果浏览器仍被系统代理或 PAC 接管，依赖高校 IP 认证的网站可能还需要加入客户端的系统代理绕过清单。
2. 直连不会自动授予学校数据库权限，下载权限仍取决于当前出口 IP 是否在学校授权范围内。
3. 不修改代理节点、订阅地址、密码、Token、硬件驱动或无关系统设置。

## 在 Codex 中使用

使用 `$proxy-rules`，例如：

> 使用 `$proxy-rules`，把 IEEE Xplore、ScienceDirect 和我所在高校的官网设为持久化直连，同时保持 Google Scholar 走代理。

Skill 入口是 [SKILL.md](SKILL.md)，默认域名清单是 [references/default-domains.yaml](references/default-domains.yaml)，界面元数据位于 [agents/openai.yaml](agents/openai.yaml)。
