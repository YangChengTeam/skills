# YangChengTeam Skills

团队公开的 Agent Skills 目录，同时是 **Claude Code** 和 **Codex / ChatGPT 桌面端**两边的插件市场（marketplace 名都叫 `yangcheng`）。

```
.claude-plugin/marketplace.json     # Claude Code 读这个
.agents/plugins/marketplace.json    # Codex 读这个
plugins/<name>/
  plugin.json                       # 可移植插件清单（Agent Plugins 1.0.0）
  .claude-plugin/plugin.json        # Claude Code 的插件清单
  README.md                         # 这个技能的说明文档
  skills/<name>/SKILL.md            # 宿主实际加载的技能文件，两边都自动发现
```

## 技能列表

| 技能 | 说明 |
| --- | --- |
| [zhiji-setup](plugins/zhiji-setup) | 一条龙把当前目录接入智迹：创建项目、签发 MCP 令牌、写好 Agent 配置、装好上报技能 |
| [zhiji-work-report](plugins/zhiji-work-report) | 把当前项目自上次上报以来的新增工作总结成工作日志，上报到智迹，并查询 Token 用量 |

## 安装（Claude Code）

注册一次 marketplace，之后装任意技能都不用再注册：

```bash
claude plugin marketplace add YangChengTeam/skills
claude plugin install zhiji-work-report@yangcheng
```

在会话里则是 `/plugin marketplace add YangChengTeam/skills`，再 `/plugin install zhiji-work-report@yangcheng`。

装完确认：

```bash
claude plugin list
```

## 安装（Codex）

同样是注册一次 marketplace：

```bash
codex plugin marketplace add YangChengTeam/skills
codex plugin add zhiji-work-report@yangcheng
```

确认：`codex plugin list`。

**ChatGPT / Codex 桌面端**用「添加插件市场」对话框，填：

| 字段 | 值 |
| --- | --- |
| 来源 | `YangChengTeam/skills` |
| Git 引用 | `main` |
| 稀疏路径 | 留空 |

仓库很小，不需要稀疏检出。真要填就得把 `.agents/plugins`、`.claude-plugin` 和 `plugins` 三个都列上，漏一个清单就读不到。

## 安装（其他宿主）

没有插件机制的宿主，直接把 `SKILL.md` 拉到技能目录。在**项目根目录**执行：

```bash
mkdir -p .agents/skills/zhiji-work-report \
  && curl -fsSL https://raw.githubusercontent.com/YangChengTeam/skills/main/plugins/zhiji-work-report/skills/zhiji-work-report/SKILL.md \
       -o .agents/skills/zhiji-work-report/SKILL.md
```

装进 `~/.agents/skills/` 则对所有项目生效。装好后重启会话，让宿主重新扫描技能目录。

## 触发

Codex 输入 `$` 会列出可用技能；Claude Code 里插件技能带插件名前缀。也可以直接用自然语言描述任务，宿主会按技能的 `description` 自己匹配。

## 新增一个技能

1. 建目录 `plugins/<name>/`，放 `plugin.json`、`.claude-plugin/plugin.json`、`README.md` 和 `skills/<name>/SKILL.md`。
2. 在 `.claude-plugin/marketplace.json` **和** `.agents/plugins/marketplace.json` 的 `plugins` 数组里各加一项，`name` 必须和 `plugin.json` 里的 `name` 一致。漏改一个，那一边就装不到。
3. 在上面的技能列表里加一行。
4. 推送前验证两边：

```bash
claude plugin validate .
codex plugin marketplace add . && codex plugin list | grep <name>
```

`SKILL.md` 的 YAML frontmatter 至少要有 `name` 和 `description`；`description` 决定宿主在什么时候自动想起这个技能，把用户可能的说法都写进去。
