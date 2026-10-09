# 智迹工作上报

把当前项目自上次上报以来的新增工作整理成工作日志，上报到智迹，并查询 Token 用量。

## 安装

```bash
claude plugin marketplace add YangChengTeam/skills
claude plugin install zhiji-work-report@yangcheng
```

Codex 使用：

```bash
codex plugin marketplace add YangChengTeam/skills
codex plugin add zhiji-work-report@yangcheng
```

## 前置条件

本技能驱动本机 `zhiji` CLI。先完成账号登录与项目 `zhiji init`，运行 `zhiji status --dir .` 检查配置；缺少 CLI 时使用 [zhiji-setup](../zhiji-setup)。

CLI 在本机读取 Codex / Claude Code 会话，并按项目目录过滤。不需要配置 MCP。个人令牌留在 `~/.zhiji/config.json`，不要写进项目或提交记录。

## 使用

用 `$zhiji-work-report` 或「上报工作」「写今天的工作日志」「同步到智迹」触发。

必须按顺序完成两步：

1. 运行 `zhiji report --dir .`，取得增量会话、Token 统计和 `runId`。没有新增工作时如实说明，不编造上报。
2. Agent 根据采集结果写 Markdown 总结，通过标准输入交给 `zhiji submit --run-id <runId> --completion <0-100>`。

`runId` 保证幂等。上传中途失败后使用同一个运行标识重试，已上传事实不会重复计数；游标只在最终提交成功后前移。

总结说明做了什么、为什么、结果如何，按主题组织，保留失败与阻塞事实。完整工作流见 [SKILL.md](skills/zhiji-work-report/SKILL.md)。

## 用量与排查

```bash
zhiji usage --range week
zhiji usage --range month --scope project
zhiji status --dir .
```

自定义区间使用 `--range custom --from YYYY-MM-DD --to YYYY-MM-DD`，含起始日、不含结束日。采集为空时查看命令的扫描诊断，核对会话所在项目目录。账号未登录或令牌失效时，让用户在「令牌管理」签发并在本机完成 `zhiji login`。
