# 智迹仓库

通过 `zhiji repo push` 把本地 git 仓库整个推送到团队 GitLab。目标仓库不存在时，由 GitLab 的 push-to-create 在推送过程中自动创建，默认落在用户自己的命名空间下且为 private，因此不需要先去网页上建库。

需要智迹 CLI 3.4.0 或更新版本与账号登录，无需初始化项目。技能会检查 CLI，缺少时从智迹实例的 `/skills/zhiji-setup/bin/` 下载。

```bash
zhiji repo push
zhiji repo push --dir ./long-video-studio --name long-video-studio
zhiji repo push --namespace team --name shared-tools --branches all
zhiji repo search "NewServiceGateway" --project root/zhiji
zhiji repo get --project root/zhiji --path server/internal/app/service_gateway.go
zhiji repo file push --project root/zhiji --path docs/deploy.md -m "补充部署说明" --file ./deploy.md
```

GitLab 令牌由智迹代管，命令自己取用后通过 git 的 credential helper 传给子进程：令牌不出现在命令行参数里（进程表可见）、不进入输出，也不会写入 `.git/config`。只有显式传 `--remote` 时才写远端配置，且写入的地址不含令牌。

默认只推当前分支，`--branches all` 才推全部本地分支。push 只传已提交的内容，所以命令会如实提示本地还有多少处未提交改动不会被推送。目录不是 git 仓库或一个提交都没有时命令直接报错——建仓库和决定提交内容是用户的事，技能不会替用户 `git init` 或 `git commit`，推送失败也不会自作主张加 `--force`。

`zhiji repo file push` 走 GitLab 接口，在已有仓库里改单个文本文件；`zhiji repo push` 走 git 协议，传整个仓库及其提交历史。原 `zhiji code` 命令已退出。

账号绑定由管理员在网页「设置 → GitLab 管理」页统一同步；未绑定或令牌过期时命令会明确提示，技能不会尝试自行签发令牌。

安装后用 `$zhiji-repo` 或「把这个项目传到 GitLab」触发。完整工作流见 `skills/zhiji-repo/SKILL.md`；本站分发时文件位于 `/skills/zhiji-repo/SKILL.md`。
