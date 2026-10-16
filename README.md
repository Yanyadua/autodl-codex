# AutoDL Codex：GPU 服务器连接与 Codex CLI 会话恢复

**在 AutoDL 上使用 Codex CLI，遇到 SSH 连不上、代理不通、断线后不知道怎么恢复？**

这套 Codex Skills 将 AutoDL SSH 排查、按需配置代理隧道、Codex CLI 安装登录和会话恢复整理成可安装的工作流，让 Codex 按步骤帮你定位问题。由 **sg-audex** 整理。

**AutoDL GPU + OpenAI Codex CLI skills:** diagnose SSH banner timeouts and Fake-IP DNS, configure optional reverse HTTP forwarding, and recover terminal sessions with screen and autossh.

[快速安装](#安装到-codex) · [下载插件 ZIP](https://github.com/Yanyadua/autodl-codex/releases/latest) · [英文介绍](docs/README.en.md) · [反馈问题](https://github.com/Yanyadua/autodl-codex/issues)

## 你可能正在遇到这些问题

| 使用场景 / 搜索关键词 | 本项目怎么帮忙 |
| --- | --- |
| AutoDL SSH 连不上、`Connection timed out during banner exchange` | 区分 DNS Fake-IP、SSH 握手与认证问题 |
| 本地 Codex 能用，AutoDL 上请求超时 | 按本地代理 → SSH 反向隧道 → 远程代理 → 登录状态的顺序排查 |
| SSH 断开后，想找回 Codex CLI 会话 | 使用 screen 保持进程，结合 `codex resume --last` 恢复历史 |
| `already has an active writer`、会话无法恢复 | 先找持有会话的进程，避免直接删除锁文件 |

适合在 AutoDL 上做深度学习、运行 GPU 实验，同时想使用 Codex 辅助编程的用户。

## 包含的技能

| 技能 | 用途 |
| --- | --- |
| `autodl-ssh` | SSH banner 超时、Fake-IP DNS、密钥检查、保活及连接复用诊断 |
| `autodl-codex` | 按需建立 HTTP 代理反向隧道，安装与登录 Codex CLI，screen 会话保持及恢复 |

适用经验环境：macOS + FlClash + AutoDL Linux。其他操作系统及代理客户端需要适配，尚未实机验证。

## 安装到 Codex

在 Codex 中粘贴下面的提示词，安装两个技能：

```text
使用 $skill-installer，从 https://github.com/Yanyadua/autodl-codex 安装 skills/autodl-ssh 和 skills/autodl-codex 两个技能。
```

也可将本仓库 `skills/` 中的两个文件夹完整复制到 `~/.agents/skills/`。如未出现新技能，重启 Codex。

### 作为插件安装

本仓库提供本地插件市场。使用支持插件市场的 Codex CLI 添加来源：

```bash
codex plugin marketplace add Yanyadua/autodl-codex
```

然后在支持的 Codex 插件界面选择此来源并安装 **AutoDL Codex**。添加来源本身不等于完成安装。需要本地终端执行能力；单纯网页聊天不能建立本地 SSH 隧道。

## 使用示例

- “帮我连接自己的 AutoDL 实例并配置 Codex CLI。”
- “AutoDL SSH 卡在 banner exchange，帮我排查。”
- “SSH 断线后恢复 Codex CLI 会话。”

首次使用需要当前实例的 SSH 主机、端口、用户名、已有密钥路径或交互式认证方式、远程工作目录；按需提供本地 HTTP/mixed 代理端口。不要把密码、私钥、API key、代理订阅或 auth.json 发进聊天。

脚本不会自动安装、自动登录或自动复制到服务器。SSH 检查脚本在本地运行；`codex-autodl` 启动器在配置好代理的远程 Linux 上运行。详细步骤在技能文件中。

## 会话保持的边界

`screen` 保持远程终端进程；`autossh` 尝试恢复反向代理隧道。本地电脑休眠、代理关闭、网络中断或 GPU 实例停止仍会影响服务，不能保证永久在线。已有直接联网方式时可以跳过代理。

## 下载与验证

[Releases](https://github.com/Yanyadua/autodl-codex/releases) 提供插件 ZIP。该包仅包含技能、脚本及图标，没有 MCP 服务或遥测。

本地完成 22 项检查，包含 Bash 语法、参数拒绝、模拟代理失败及 SSH 复用流程。未在本次发布中连接真实 AutoDL 实例或测试长期在线。详见 [验证报告](docs/validation.md)。发布到 GitHub 不代表已通过 OpenAI 官方目录审核。

## 常见问题

**这是自动购买或启动 AutoDL 显卡的工具吗？**

不是。你需要自己的 AutoDL 实例和 Codex 账号；项目提供连接、配置及排障技能。

**不用 FlClash 能使用吗？**

SSH 诊断可以独立使用。代理工作流需要本地可用的 HTTP/mixed 代理；已有直接联网方式时可以跳过代理。其他客户端需要按实际端口适配。

**关闭电脑后远程 Codex 还能继续工作吗？**

依赖本地代理的网络请求会中断。screen 保持进程，不能替代持续可用的网络或正在运行的 GPU 实例。

**支持 Windows 吗？**

目前经验环境是 macOS + AutoDL Linux，尚未验证 Windows，欢迎附脱敏信息反馈适配结果。

## 分享给需要的人

可以直接分享仓库链接和这段简介：

> 在 AutoDL 上用 Codex CLI，经常遇到 SSH banner 超时、代理连接失败或断线恢复问题？AutoDL Codex 把连接诊断、代理隧道和会话恢复整理成两个可安装的 Codex Skills，附中文安装说明。https://github.com/Yanyadua/autodl-codex

## 反馈

请通过 [GitHub Issues](https://github.com/Yanyadua/autodl-codex/issues) 报告问题，附系统版本、Codex 版本和已脱敏错误信息。

社区独立项目，与 AutoDL 或 OpenAI 无隶属关系。当前未授予额外开源许可证。
