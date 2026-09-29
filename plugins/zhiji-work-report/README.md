# 智迹工作上报

把当前项目自上次上报以来的新增工作，整理成一份工作日志上报到智迹；顺带支持查询 Token 用量。

## 安装

```bash
claude plugin marketplace add YangChengTeam/skills
claude plugin install zhiji-work-report@yangcheng
```

marketplace 只需注册一次。装好后重启会话，让宿主重新扫描技能目录，用 `claude plugin list` 确认。

Codex 不认 Claude Code 的插件机制，直接拉文件到技能目录（在**项目根目录**执行）：

```bash
mkdir -p .codex/skills/zhiji-work-report \
  && curl -fsSL https://raw.githubusercontent.com/YangChengTeam/skills/main/plugins/zhiji-work-report/skills/zhiji-work-report/SKILL.md \
       -o .codex/skills/zhiji-work-report/SKILL.md
```

## 前置条件

技能本身**不带凭据、不读文件**，它只是驱动本机的 `zhiji` MCP 服务。所以当前项目必须先接入 MCP：

1. 打开智迹控制台的「MCP 接入」页，为这个项目签发令牌。
2. 按页面给出的片段配置 Codex 或 Claude Code。
3. 重启会话。

MCP 服务跑在本机，读取的是本机的 Codex / Claude Code 会话文件——只有本机能读到这些内容，远端服务器做不到。

工具名以 `zhiji_` 开头，不同宿主可能带前缀（例如 `mcp__zhiji__zhiji_report_work`）。如果看不到这些工具，说明 MCP 还没配好。

## 怎么用

直接说就行：

- 「上报工作」
- 「总结这次工作并上报」
- 「写今天的工作日志」
- 「同步到智迹」

也可以主动触发：Codex 里输入 `$` 选 `zhiji-work-report`。

## 上报流程

技能约束模型走完两步，缺一不可：

1. **采集** —— 调用 `zhiji_report_work`，拿回自上次上报以来的新增会话原文摘录、Token 统计和一个 `runId`。
   没有新增内容时它会明确说明，这时应当直接回「本次没有新增工作」，**不编造内容去上报**。
2. **上报** —— 读完采集结果后自己写一份 Markdown 总结，调用 `zhiji_submit_work` 提交：
   - `runId`：上一步返回的，原样传回
   - `summary`：Markdown 总结
   - `items`：要点列表，用于列表页展示；不传会从 Markdown 自动提取
   - `completion`：完成度百分比（0-100）

`runId` 保证幂等：重复提交同一个运行标识只会更新那一条记录，网络重试不会重复计数。

## 总结写成什么样

技能里写死了要求，是写给同事看的，不是写给模型看的：

- 说清**做了什么、为什么做、结果如何**，而不是罗列调用过哪些工具
- 按主题分组，不要按时间流水账；改了代码就点出关键文件和行为变化
- 保留事实：测试失败、功能未完成、被阻塞的部分都要写出来，不只报喜
- 中文，十几行以内，细节放要点里

## 查询用量

问「本周用了多少 token」「这个月的用量」时会调用 `zhiji_usage`：

- `range`：`today` / `week` / `month` / `all` / `custom`（`custom` 再传 `from`、`to`，格式 `YYYY-MM-DD`，含头不含尾）
- `scope`：`member` 只看自己，`project` 看整个项目

回答会给出总量和模型分布，不会把原始 JSON 贴出来。

## 排查

| 现象 | 原因 |
| --- | --- |
| 工具报未配置或 401 | 令牌无效或已被删除，去「MCP 接入」重新签发 |
| 采集为空但确信有工作 | MCP 按项目根目录过滤会话，确认会话确实发生在**当前项目目录**下 |
| 宿主看不到 `zhiji_` 工具 | MCP 没配好，或者配完没重启会话 |
