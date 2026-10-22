# 智迹团队文档

通过 `zhiji doc` 检索、阅读和撰写团队 GitLab Wiki。智迹代为保管每人的 GitLab 令牌，读写都以用户本人的身份进行，权限由 GitLab 判定。

需要智迹 CLI 2.7.0 或更新版本与账号登录，无需初始化项目。技能会检查 CLI，缺少时从智迹实例的 `/skills/zhiji-setup/bin/` 下载。

```bash
zhiji doc search "增量上报 游标"
zhiji doc list --project root/zhiji
zhiji doc get  --project root/zhiji --page 接入说明
zhiji doc push --project root/zhiji --title "发布流程" --file ./release.md
```

检索按项目逐个进行，命令会打印搜索了多少个项目，无法搜索的项目连同原因输出到标准错误——**搜不到和没搜成是两回事**，技能要求 Agent 如实说明覆盖范围。`--project` 接受数字 ID 或完整路径，名称不唯一时列出候选并报错，不会替用户挑一个。

写入前应先检索同主题文档，已存在就带 `--slug` 更新而不是新建。正文为 UTF-8 文本，单次最多 1 MB；写入会以用户本人的身份记录在 GitLab 提交历史中。

账号绑定由管理员在网页「GitLab 账号」页统一同步；未绑定或令牌过期时命令会明确提示，技能不会尝试自行签发令牌。

安装后用 `$zhiji-doc` 或「查一下团队文档」触发。完整工作流见 `skills/zhiji-doc/SKILL.md`；本站分发时文件位于 `/skills/zhiji-doc/SKILL.md`。
