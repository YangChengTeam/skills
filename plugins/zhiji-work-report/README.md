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

本技能驱动本机 `zhiji` CLI **2.4.0 或更新版本**。先完成账号登录与项目 `zhiji init`，运行 `zhiji status` 检查配置；缺少 CLI 时使用 [zhiji-setup](../zhiji-setup)。2.3.x 可用 `zhiji upgrade` 更新。

CLI 在本机读取 Codex / Claude Code 会话，并按项目目录过滤。不需要配置 MCP。个人令牌留在 `~/.zhiji/config.json`，不要写进项目或提交记录。

## 使用

用 `$zhiji-work-report` 或「上报工作」「写今天的工作日志」「同步到智迹」触发。

先采集，再按业务范围决定是否上报：

1. 运行 `zhiji report`，取得增量会话、Token 统计和 `runId`。没有新增工作时如实说明，不编造上报。
2. 确认完整批次仅为智迹安装、登录、接入配置、技能安装或上报验证时，执行
   `zhiji skip --run-id <runId> --reason onboarding-only`，只推进游标，不上传事实或创建日志。
3. 有业务工作时，Agent 写 Markdown 总结，通过标准输入交给 `zhiji submit --run-id <runId> --completion <0-100>`。
   混合批次的总结省略接入事项，事实保持完整；智迹产品本身的开发、修复和部署仍是业务工作。
   无正文、截断内容或仅有文件变更数量不足以判断纯接入，不得据此 skip。

`runId` 保证上报幂等。上传中途失败后使用同一个运行标识重试，已上传事实不会重复计数；游标在最终提交或 skip 确认后前移。已上传部分事实的批次不能 skip。

总结说明做了什么、为什么、结果如何，按主题组织，保留失败与阻塞事实。完整工作流见 [SKILL.md](skills/zhiji-work-report/SKILL.md)。

## 用量与排查

```bash
zhiji usage --range week
zhiji usage --range month --scope project
zhiji status --dir .
```

自定义区间使用 `--range custom --from YYYY-MM-DD --to YYYY-MM-DD`，含起始日、不含结束日。采集为空时查看命令的扫描诊断，核对会话所在项目目录。账号未登录或令牌失效时，让用户在「令牌管理」签发并在本机完成 `zhiji login`。
