# Magpie：把 Google 账号接入 Claude 与 Codex

这是一份操作文档（也可作为可选 Skill），说明如何把本机用官方 `agy` 登录的 Google 账号所开放的模型，经 Magpie 接到 Claude Desktop、Claude Code 和 Codex。它不是 Magpie、Google、Anthropic 或 OpenAI 的官方产品，也不包含这些上游软件、账号或凭据，更不能让任意 Google 账号绕过订阅、地区、验证或额度限制。

## 它怎么工作

```text
Google 账号
  → 本机官方 agy（由社区 agy-cli 插件调用）
  → 本机 Magpie 网关
  → Claude Desktop／Claude Code，或 Codex（CLI／应用）
```

登录在官方 Google 页面完成，请求走你自己电脑上的 Magpie 网关，不经过本仓库。两条客户端分支的 API 不同，必须分开配置、分开验收、分开回滚：

- Claude Desktop／Claude Code 走 Anthropic Messages
- Codex（CLI／应用）走 OpenAI Responses（经 Magpie 网关翻译）

配好 Claude 不等于 Codex 已可用，反之亦然。本文也不说明如何把模型直连到未列出的其他客户端。

## 两种角色

- **执行代理**：正在阅读或加载本指南、协助检查和操作的 AI 代理。任何能读 Markdown 的代理都可以按文档执行，或逐步指导你动手；有终端或文件权限的，可在你同意后运行文中已写明的命令。不保证所有代理都能自动跑完。
- **被配置的客户端**：Claude Desktop、Claude Code、Codex。即使执行代理碰巧是 Codex，也不等于已经把 Codex 配成使用 Google 账号模型。把本 Skill 放进 Codex 的 skills 目录，只是让它作为执行代理来加载文档，并不是接入本身。

## 开始之前

在目标电脑上自行准备（本仓库不提供安装包）：

- [Magpie](https://usemagpie.ai/)
- 本机官方 [Antigravity CLI（agy）](https://github.com/google-antigravity/antigravity-cli)（缺失时，以插件文档为准，可在登录阶段按官方方式安装）
- 社区插件 `@magpie-community/opencode-agy-cli-auth`（按 [setup.md](magpie-google-agent-setup/references/setup.md) 安装；不要使用已退役的「在 Magpie 里直接登录 Antigravity」）
- 你要接入的客户端：Claude Desktop、Claude Code 和／或 Codex
- 一个由你本人在官方页面完成登录的 Google 账号

不是所有 Google 账号都能用。能否调用、能用哪些模型，以该账号在 Magpie／agy 里实际开放的列表为准，不要猜测型号。登录与验证必须由账号本人完成，不要把密码、验证码或令牌发给仓库作者，或写进本仓库。

## 最短使用路径

1. 打开 [SKILL.md](magpie-google-agent-setup/SKILL.md)，按其中的角色、权限和停点操作。没有 Skill 加载器时，直接把它当作普通 Markdown 阅读即可。
2. 按 [setup.md](magpie-google-agent-setup/references/setup.md) 检查环境、安装插件并登录账号，用 `magpie models` 记下实际模型 id。
3. 只配置你需要的客户端，不要互相套用配置：
   - [接入 Claude Desktop／Claude Code](magpie-google-agent-setup/references/clients-claude.md)
   - [接入 Codex CLI／应用](magpie-google-agent-setup/references/clients-codex.md)
4. 在对应客户端里做真实的文字和工具测试。一条分支通过，不能宣布另一条也完成。
5. 遇问题见 [troubleshooting.md](magpie-google-agent-setup/references/troubleshooting.md)。从旧 WSL 账号池或旧文档目录名迁移，见 [migration.md](magpie-google-agent-setup/references/migration.md)。架构与上游依据见 [architecture.md](magpie-google-agent-setup/references/architecture.md)。

若所用代理支持 Skill：把仓库里的 `magpie-google-agent-setup` 文件夹，按该代理自己的方式放入技能目录。目标位置已有同名目录，或旧名 `magpie-local-account-pool`、`magpie-claude-multi-account` 时，不要覆盖，用新名并存后再由你决定如何处理旧副本。本包不罗列其他产品未经查证的安装路径。

仅当执行代理是 Codex、并且要加载本 Skill 时，才使用 `~/.codex/skills/`（Windows 一般为 `%USERPROFILE%\.codex\skills\`）和 [agents/openai.yaml](magpie-google-agent-setup/agents/openai.yaml)。这与上面第 3 步的「把 Codex 配成目标客户端」不是同一步骤。

## 仓库目录

```text
README.md
magpie-google-agent-setup/
  SKILL.md                 通用入口（也可被 Skill 加载）
  agents/openai.yaml       可选：仅给 Codex 当执行代理时的界面元数据
  references/
    setup.md               安装插件与登录账号
    clients-claude.md      接入 Claude
    clients-codex.md       接入 Codex
    troubleshooting.md     排障
    migration.md           旧环境与旧目录名
    architecture.md        上游依据与核实结论
```

本仓库只有文档和上述元数据，没有程序、运行配置或登录记录。

## 支持状态与来源

步骤依据 **2026-11-24** 查阅的上游文档整理，结论和链接写在 [architecture.md](magpie-google-agent-setup/references/architecture.md)。整理时没有在各种系统／版本上做实测，也没有完成 Codex + agy-cli 的端到端验收。

就文档而言：Claude 与 Codex 两条分支都有 Magpie 官方说明可走本机网关。Codex 只接受 Responses API，而 agy-cli 插件本身接收 Anthropic Messages，因此 Codex 路径依赖 Magpie 的翻译，属于有依据的条件支持，不是「已全面验证」。插件对部分 Codex 提示词的改写、以及 Codex 自带工具是否全部可用，都需要你在本机验收后再下结论。

## 账号风险与隐私

- 上游说明：不要在 Magpie 里直接登录 Antigravity（已退役，存在封号风险）。应使用 agy-cli 插件，由本机 `agy` 发出请求。即便如此，插件和本文都不能保证账号不受 Google 限制。
- 本文不能绕过订阅、额度、地区或验证要求，也不保证所有账号或所有模型都能用。
- 不要把 Magpie 配置、OAuth 链接、账号目录、网关 Key、`config.toml`、日志或截图提交到本仓库或公开 issue。反馈问题时只提供脱敏错误和版本。

## 许可证

本仓库尚未指定许可证，请勿假定可以任意复制或再分发。使用前请等待作者补充许可；上游项目（Magpie、agy、各客户端等）的许可不自动覆盖此文档包。
