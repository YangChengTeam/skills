# YangChengTeam Skills

团队公开的 Agent Skills 目录，适用于 **Claude Code** 和 **Codex**。

每个技能是 `skills/<slug>/` 下的一个目录：`SKILL.md` 是宿主实际加载的技能文件，`README.md` 是这份技能的说明文档。

## 技能列表

| 技能 | 说明 | 安装 |
| --- | --- | --- |
| [zhiji-work-report](skills/zhiji-work-report) | 把当前项目自上次上报以来的新增工作总结成工作日志，上报到智迹，并查询 Token 用量 | `npx skillstore add YangChengTeam/zhiji-work-report` |

## 安装

```bash
npx skillstore add YangChengTeam/<slug>
```

在**项目根目录**执行。CLI 会自动识别 Codex 与 Claude Code 的技能目录，两个都存在时一起装。装好后重启会话，让宿主重新扫描技能目录。

没有 npx，或者网络装不上时，直接从仓库拉文件：

```bash
mkdir -p .claude/skills/<slug> .codex/skills/<slug> \
  && curl -fsSL https://raw.githubusercontent.com/YangChengTeam/skills/main/skills/<slug>/SKILL.md \
       -o .claude/skills/<slug>/SKILL.md \
  && cp .claude/skills/<slug>/SKILL.md .codex/skills/<slug>/SKILL.md
```

## 触发

Codex 输入 `$` 会列出可用技能；Claude Code 同样能按名字调用。也可以直接用自然语言描述任务，宿主会按技能描述自己匹配。

## 新增一个技能

1. 建目录 `skills/<slug>/`，写 `SKILL.md`（YAML frontmatter 至少要有 `name` 和 `description`）和 `README.md`。
2. 在上面的技能列表里加一行。

`description` 决定宿主在什么时候自动想起这个技能，把用户可能的说法写进去。
