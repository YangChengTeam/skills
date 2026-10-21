# 智迹工单协作

通过 `zhiji ticket` 把跨团队需求派到负责组，认领处理并记录进展，最后由发起人验收。

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
claude plugin install zhiji-ticket@yangcheng
```

Codex 使用：

```bash
codex plugin marketplace add YangChengTeam/skills
codex plugin add zhiji-ticket@yangcheng
```

需要本机智迹 CLI 与账号登录。工单是账号级能力，任何目录都能调用，无需初始化项目；缺少 CLI 时使用 [zhiji-setup](../zhiji-setup)。

## 使用

用 `$zhiji-ticket` 或「提个工单」「看看我的工单」「有没有待我处理的工单」「验收一下」触发。

```bash
zhiji ticket groups
zhiji ticket list
zhiji ticket show <工单ID>
```

Agent 先阅读各组职责，再根据需求选组，并在信息不明确时与用户核对。创建时直接指定 `--group`；名称缺失或有歧义会报错并列候选组，不猜测派单。

流程是：发起人选组提交 → 组内成员认领 → 处理并记录进展 → 提交验收 → 发起人确认关闭。处理人不能关闭自己处理的工单。验收打回必须说明问题，工单会回到处理中。

完整命令、状态与权限说明见 [SKILL.md](skills/zhiji-ticket/SKILL.md)。
