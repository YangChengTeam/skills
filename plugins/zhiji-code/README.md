# 智迹团队代码

通过 `zhiji code` 跨项目搜索、阅读和提交团队 GitLab 仓库里的文件。智迹代为保管每人的 GitLab 令牌，读写都以用户本人的身份进行，权限由 GitLab 判定。

需要智迹 CLI 2.7.0 或更新版本与账号登录，无需初始化项目。技能会检查 CLI，缺少时从智迹实例的 `/skills/zhiji-setup/bin/` 下载。

```bash
zhiji code projects
zhiji code search "func NewServiceGateway" --project root/zhiji
zhiji code get  --project root/zhiji --path server/internal/app/service_gateway.go
zhiji code push --project root/zhiji --path docs/deploy.md -m "补充部署说明" --file ./deploy.md
```

检索按项目逐个进行，命令会打印搜索了多少个项目，无法搜索的项目连同原因输出到标准错误。这是基础搜索（相当于 `git grep`）而非语义检索，关键词应是实际出现在代码中的标识符。

`code push` 直接写入远端分支，**没有 code review 环节**，所以技能要求 Agent 先读取现有内容再改、写清提交说明、并确认目标分支。内容为 UTF-8 文本，单次最多 1 MB，省略 `--branch` 时提交到默认分支。提交作者记为用户本人。二进制、批量改动、重命名和删除不走这里，应直接使用 git。

账号绑定由管理员在网页「GitLab 账号」页统一同步；未绑定或令牌过期时命令会明确提示，技能不会尝试自行签发令牌。

安装后用 `$zhiji-code` 或「搜一下团队代码」触发。完整工作流见 `skills/zhiji-code/SKILL.md`；本站分发时文件位于 `/skills/zhiji-code/SKILL.md`。
