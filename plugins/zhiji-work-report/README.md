# 智迹工作上报

把当前项目自上次上报以来的新增工作整理成工作日志，上报到智迹，并查询 Token 用量。

## 实例迁移

团队生产地址已迁移到 `http://172.16.6.163:8090`。保存的旧地址为
`http://172.16.6.139:8089` 时，先为当前终端设置
`ZHIJI_API_URL=http://172.16.6.163:8090`，再运行 `zhiji upgrade`
和 `zhiji skills update`。升级不会自动修改账号保存的地址；永久迁移时先备份
`~/.zhiji/config.json`，仅更新 `apiUrl`，保留原 PAT、项目 ID 和游标。
其它自定义实例保持原样。登录显式使用 `--url <实例地址>`。
插件安装副本通过宿主插件市场更新，CLI 只更新项目内副本；更新后重新读取技能。

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

本技能驱动本机 `zhiji` CLI **2.6.0 或更新版本**。先完成账号登录与项目 `zhiji init`，运行 `zhiji status` 检查配置；缺少 CLI 时使用 [zhiji-setup](../zhiji-setup)。2.3.x–2.5.x 可用 `zhiji upgrade` 更新。

旧项目运行一次 `zhiji skills update --auto on` 更新并接管已安装副本，之后 report 前
每天自动检查一次，断网继续采集，人工修改保留并提示。新 init 安装的副本默认受管。
插件缓存仍由宿主市场管理，CLI 不改写插件缓存；技能更新后需要重新读取指令。

历史补采使用 `zhiji report --history` 建立或继续计划，支持活动/归档及 .jsonl.zst。
每批只含一个会话，长会话分段，最多 24000 个展示字符、1000 条事实或约 512 KiB。
每批单独审阅总结并 submit/skip，再运行相同 report 继续；仅显示准备进度时继续命令。
未完成批次保留同一 runId，submit 可使用已保存总结重试；服务端回执恢复丢失响应的计数。
`--history --status` 查看进度，`--history --new` 新建扫描并保留旧计划。正常游标不变，
已存在事实不重复入库，待处理历史批次不自动过期。总结标明历史时间，不冒充今天新成果。

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
