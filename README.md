# OpenWrt sing-box Gateway

一个面向 OpenWrt 的 LuCI 管理界面项目，计划使用 sing-box 为家庭网络提供可配置的策略路由，并支持 Hysteria2 出站。

> **项目状态：规划阶段，尚无可安装的软件包。**
> 本仓库目前记录项目目标和设计方案。下文中的功能均为计划能力，不代表已经实现或验证。请勿将它部署到承载家庭网络的路由器上。

[English](#english)

## 为什么做这个项目

在 OpenWrt 上运行代理和分流，通常需要手工维护多个部分：sing-box 配置、规则集、DNS、定时更新脚本、防火墙转发和服务启动顺序。出现问题时，还要自己从日志里判断是规则、DNS、隧道还是防火墙造成的。对家庭网络来说，一处配置错误就可能影响多台设备上网。

这个项目希望把这些环节整理成 OpenWrt 原生的 LuCI 管理体验：配置有明确入口，应用前能校验，规则更新失败时保留旧版本，代理或服务故障时尽量恢复普通直连，并能从页面查看状态和诊断信息。

项目来自一个实际家庭网络的 sing-box + Hysteria2 分流使用场景，也希望整理成他人可以审阅、复现和改进的开源实现。

## 计划实现什么

### sing-box 服务管理

- 查看 sing-box 是否安装、版本和运行状态。
- 在 LuCI 中启停服务和设置开机启动。
- 生成 sing-box 配置，并在应用前进行语法校验。
- 展示必要的运行日志和错误原因。

### Hysteria2 出站配置

- 配置服务器地址、端口、认证、TLS/SNI 和 Salamander 混淆参数。
- 配置 Brutal 拥塞控制带宽参数，并清楚标明单位。
- 对密码等敏感字段进行遮蔽，诊断信息不包含密钥。

### 策略路由和分流

- 按域名、IP/CIDR 等规则将流量分配到代理、直连或拒绝策略。
- 显示规则匹配顺序，并支持用户添加自己的规则。
- 从 Shadowrocket 规则源获取规则，在本机转换为 sing-box 支持的规则集；sing-box 是运行核心，Shadowrocket 仅是可选的规则数据来源。
- 自定义规则与下载的上游规则分开保存，更新上游规则时不覆盖用户设置。
- 明确默认策略和代理不可用时的行为，避免用户误以为“直连回退”仍然保护代理隐私。

### 规则更新和回滚

- 支持手动更新和可配置的定时更新。
- 更新流程计划包括下载、解析、格式转换、编译、校验和原子替换。
- 遇到网络错误、未知规则语法或校验失败时，保留当前可用规则并报告原因。
- 保留前一份有效规则和配置，必要时回滚。

### DNS、状态和诊断

- 展示 DNS 接管方式及代理相关配置，避免 DNS 环路。
- DNS 和 IPv6 通告设置单独控制，不在安装时隐式覆盖现有网络配置。
- 提供服务、隧道和规则状态，以及脱敏诊断信息导出。

### 安全应用和恢复

- 安装后不自动接管 LAN 流量；由用户明确启用。
- 在配置或规则校验通过后才应用。
- 计划支持服务退出后的转发清理、配置回滚和清晰的停用流程。
- 插件卸载默认不删除用户配置和备份。

## 设计原则

1. **安全默认值**：安装和升级不应悄悄改变全 LAN 网络路径。
2. **失败可恢复**：规则或配置更新失败时，当前可用版本继续工作。
3. **配置可解释**：用户能看见规则来源、匹配顺序、更新结果和故障模式。
4. **凭据最小暴露**：不把节点密码写进日志、诊断包或版本控制。
5. **遵循 OpenWrt 机制**：优先使用 UCI、procd、rpcd/ubus、LuCI 和系统防火墙集成。
6. **上游与个人规则分离**：上游规则可更新，个人例外独立保留。

## 计划中的工作方式

项目计划分阶段推进：

1. 明确 OpenWrt、sing-box 和防火墙兼容范围。
2. 建立软件包骨架、LuCI 菜单和只读状态页。
3. 实现 Hysteria2 配置、配置生成和校验。
4. 实现 Shadowrocket 常见规则语法适配和用户自定义规则。
5. 实现安全更新、状态显示和回滚。
6. 在隔离测试设备验证后，再开发 LAN 接管和故障恢复。
7. 完成安装、升级、卸载、恢复文档及软件包发布。

详细范围与阶段验收见 [PROJECT_PLAN.md](PROJECT_PLAN.md)。

## 当前状态和贡献

目前项目处于规划阶段，功能、配置格式、兼容矩阵和安装包尚未发布。暂时没有可用于生产路由器的安装步骤。欢迎通过 GitHub Issues 讨论需求、OpenWrt 版本覆盖和设计问题；开始接受代码贡献前，会补充许可证、贡献指南和安全问题报告方式。

如果你准备贡献代码，请先开 Issue 讨论较大的功能或架构变更。涉及防火墙、默认路由、DNS 或自动接管的改动，需要包含失败场景与恢复方案。

## 许可证和上游规则说明

项目许可证尚待确定，当前不应将本仓库内容视为已明确授权的开源软件。

计划支持从 [Johnshall/Shadowrocket-ADBlock-Rules-Forever](https://github.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever) 获取规则。该项目规则有自己的许可证和署名要求；本项目计划在运行时下载并本机转换，并在界面和文档中标明来源。规则适配器不会改变上游项目的所有权或维护责任。

---

## English

OpenWrt sing-box Gateway is a planned LuCI application for managing sing-box policy routing on OpenWrt, with Hysteria2 outbound support.

The project aims to make configuration, rule updates, diagnostics, and recovery easier to understand and safer for home networks. Planned capabilities include LuCI-based service management, Hysteria2 configuration, policy routing, local custom rules, conversion of supported Shadowrocket rule syntax into sing-box rule sets, validated updates with rollback, and diagnostics.

**Status: planning only. There is no installable package yet.** Features described above are proposed, not implemented or verified. See [PROJECT_PLAN.md](PROJECT_PLAN.md) for scope and milestones. The project license and supported-version matrix have not been finalized.
