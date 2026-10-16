# 本地验证报告

- 通过：Manifest displayName
- 通过：Manifest shortDescription
- 通过：Manifest longDescription
- 通过：Manifest developerName
- 通过：Publisher identity sg-audex
- 通过：Asset logo
- 通过：Asset composerIcon
- 通过：Skill frontmatter autodl-ssh
- 通过：Skill frontmatter autodl-codex
- 通过：Skill reference references/setup.md
- 通过：Bash syntax autodl-ssh-check
- 通过：Help autodl-ssh-check
- 通过：Bash syntax codex-autodl
- 通过：Help codex-autodl
- 通过：Reject remote-command injection
- 通过：Launcher passes proxy, directory, session and Codex arguments
- 通过：Failed proxy prevents launching screen
- 通过：Reject out-of-range port
- 通过：Mocked fresh and reused SSH workflow
- 通过：Fresh connections isolated and master options take precedence
- 通过：No original personal endpoints or local paths
- 通过：ZIP integrity and root manifest

限制：未连接实际 AutoDL 实例，未进行真实登录、网络长时间稳定性或平台上传扫描。SSH 和启动器行为使用隔离的命令替身验证，不能代替实机测试。技能内流程来自现有使用经验。内置 quick_validate.py 因环境缺少 PyYAML 无法运行，本报告使用独立的限定格式检查验证前言；未冒称官方验证器通过。包的字段按官方文档逐项核对，最终可提交性由平台扫描决定。
