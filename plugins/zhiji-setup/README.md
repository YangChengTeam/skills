# zhiji-setup

一条龙把当前目录接入智迹：创建或复用项目、安装技能，并配置本地 CLI。需要智迹 CLI 3.4.0 或更新版本。

`zhiji init` 安装四个项目技能：`zhiji-work-report`、`zhiji-ticket`、`zhiji-repo` 和 `zhiji-doc`。共享知识库通过 `zhiji doc` 明确检索，GitLab 仓库与代码通过 `zhiji repo` 操作。项目 Wiki 仍位于网页「代码库」，与共享知识库独立。

`.env.zhiji` 可能包含本人的 GitLab 令牌。`init` 会先把它加入 `.gitignore`；不要提交或打印这个文件。智迹账号令牌保存在 `~/.zhiji/config.json`。

## 安装

```bash
claude plugin marketplace add YangChengTeam/skills
claude plugin install zhiji-setup@yangcheng
```

Codex 使用 `codex plugin marketplace add YangChengTeam/skills` 和 `codex plugin add zhiji-setup@yangcheng`。安装后说「接入智迹」或触发 `$zhiji-setup`。具体步骤见 [SKILL.md](skills/zhiji-setup/SKILL.md)。

技能文本在本仓库；CLI 二进制由智迹实例的 `/skills/zhiji-setup/bin/` 提供。默认实例为 `http://172.16.6.163:8090`，用户明确指定其它实例时按其要求。
