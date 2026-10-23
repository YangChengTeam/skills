---
name: zhiji-repo
description: 管理团队 GitLab 仓库：推送本地仓库、搜索和读取代码、提交单文件改动。当用户说「把项目传到 GitLab」「搜索团队代码」「读仓库文件」「提交单个文件」时使用。
---

# 智迹仓库推送

用 `zhiji repo push` 把本地 git 仓库推到团队 GitLab。目标仓库不存在时由 GitLab 的 push-to-create 在推送过程中自动建出来，默认落在你自己的命名空间下且为 private。推送以**你本人的身份**进行，权限由 GitLab 判定。不要索要或回显任何令牌。

**这是智迹唯一的建库方式。** 没有任何接口能凭空建出一个 GitLab 仓库——建库就是往一个不存在的
地址推送。所以当用户要把东西存进一个还不存在的仓库时，答案永远是「在本地建仓库、提交、
`zhiji repo push`」，而不是「找个名字最像的现有仓库提交进去」。

跑过 `zhiji init` 的目录里，令牌、GitLab 地址和你的命名空间都已经在 `.env.zhiji` 里，
这条命令全程不向智迹服务端要任何东西；没跑过 `init` 也能用，那时它会回退到接口去取。

## 实例地址与迁移

团队生产实例为 `http://172.16.6.163:8090`，旧实例为 `http://172.16.6.139:8089`。
用户明确指定其它实例时按其要求。否则先确定当前命令的有效地址：`ZHIJI_API_URL` 优先，
项目命令再看项目 `.env.zhiji`，最后看 `~/.zhiji/config.json` 的 `apiUrl`；工单和服务命令
使用账号配置，不要求项目初始化。只检查地址字段，不输出完整账号配置或令牌。

有效地址与旧实例完全相同（忽略尾斜线）或尚未配置地址时，对本次所有 `zhiji` 进程设置
`ZHIJI_API_URL=http://172.16.6.163:8090`，包括升级、技能更新、采集和提交。其它自定义地址
保持原样，不能仅按 IP 子串替换。不要先访问旧站，失败后再把迁移问题当成令牌失效。

迁移复用原 PAT 和项目 ID，无需重签令牌、重建项目或重置游标。升级 CLI 或技能不会自动修改
保存的 `apiUrl`；用户要求永久迁移时，先备份再只更新该地址字段，保留令牌和其它字段。
`zhiji login` 必须显式带 `--url <本次确定的实例地址>`，它不读取 `ZHIJI_API_URL` 来选择地址。

## 准备 CLI

先找 PATH 中的 `zhiji` 或本 SKILL.md 所在目录的 `bin/zhiji`（Windows 为 `bin/zhiji.exe`），运行 `version`。需要 **3.4.0 或更新版本**；版本过低或不存在时，下载到本技能的 `bin/`，不要覆盖系统中的旧文件。

智迹地址默认 `http://172.16.6.163:8090`。用户指定的实例优先，其次参考 `~/.zhiji/config.json` 的 `apiUrl`。二进制统一从该实例的 `/skills/zhiji-setup/bin/` 下载：

| 系统 | 文件名 |
| --- | --- |
| Windows x86_64 | `zhiji-windows-amd64.exe` |
| macOS Apple 芯片 | `zhiji-darwin-arm64` |
| macOS Intel | `zhiji-darwin-amd64` |
| Linux x86_64 | `zhiji-linux-amd64` |
| Linux arm64 | `zhiji-linux-arm64` |

macOS / Linux 示例（将技能目录替换为实际绝对路径）：

```bash
ZHIJI_BASE=http://172.16.6.163:8090
ZHIJI_DIR="<技能目录>"
mkdir -p "$ZHIJI_DIR/bin"
curl -fsSL "$ZHIJI_BASE/skills/zhiji-setup/bin/zhiji-$(uname -s | tr 'A-Z' 'a-z')-$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')" -o "$ZHIJI_DIR/bin/zhiji"
chmod +x "$ZHIJI_DIR/bin/zhiji"
```

Windows PowerShell 示例：

```powershell
$zhijiBase = 'http://172.16.6.163:8090'
$zhijiDir = '<技能目录>'
New-Item -ItemType Directory -Force "$zhijiDir\bin" | Out-Null
Invoke-WebRequest "$zhijiBase/skills/zhiji-setup/bin/zhiji-windows-amd64.exe" -OutFile "$zhijiDir\bin\zhiji.exe"
```

下载后再次检查版本。本文的 `zhiji` 指选定的合格二进制，必要时用完整路径执行。

若提示尚未登录，先按迁移规则核对地址；没有有效令牌时，让用户在智迹「令牌管理」签发并自己执行 `zhiji login --url <本次确定的实例地址> --token <你的令牌>`。不要索要、回显或写入项目令牌。

## 推送仓库

```bash
zhiji repo push
zhiji repo push --dir ./long-video-studio --name long-video-studio
zhiji repo push --namespace team --name shared-tools --branches all
```

默认推当前目录所在仓库的**当前分支**，目标是 `<你的 GitLab 用户名>/<仓库目录名>`。
`--branches all` 推送全部本地分支——只在用户明确要"整个仓库搬过去"时用，不要默认替他决定。

推送前先向用户确认这三件事，它们都不可撤销：

1. **目标仓库路径**。命令会先打印"目标仓库 X 不存在，GitLab 会在推送时自动创建"或"已存在，本次是追加推送"，把这句转达给用户。
2. **推哪些分支**。当前分支还是全部分支。
3. **仓库内容**。内网 GitLab 对全体登录用户可见（实例默认可见性为 internal），不要把含密钥、客户数据或个人资料的目录推上去。推之前让用户确认仓库里没有不该外传的内容。

命令会如实提示"本地有 N 处未提交改动，本次不会被推送"——**把这句转达给用户**，因为 push 只传已提交的内容，未提交的改动留在本地。用户想连这些一起推时，让他自己先 commit，不要替他提交。

成功后输出仓库地址，把地址给用户。需要同时配置本地远端时加 `--remote <远端名>`，写入的地址不含令牌。

## 搜索、阅读与单文件提交

原 `zhiji code` 已并入 `zhiji repo`：

```bash
zhiji repo list
zhiji repo search "NewServiceGateway" --project root/zhiji
zhiji repo get --project root/zhiji --path server/internal/app/service_gateway.go
zhiji repo file push --project root/zhiji --path docs/deploy.md -m "补充部署说明" --file ./deploy.md
```

`--project` 用 GitLab 项目 ID 或完整路径；名称不唯一时命令列候选并报错，不要猜。单文件提交只适合已有仓库的文本文件：提交前先读现有文件、确认目标分支和提交内容；成规模变更走本地 git。`zhiji repo push` 仍是整个仓库的 git 推送，不能把它当成单文件提交。

## 边界

- 已有仓库里改一个文件请用 `zhiji repo file push`；它走 GitLab 接口，不需要本地 git。
- 目录不是 git 仓库、或一个提交都没有时命令会报错。**不要替用户 `git init` 或 `git commit`**：建仓库和决定提交内容是用户的事。
- 没有绑定 GitLab 时命令返回"尚未绑定"，让用户找管理员在网页「设置 → GitLab 管理」页同步一次；令牌过期同理。不要试图自己签发令牌。
- 推送失败时原样转达 git 的报错。常见原因是远端已有分支且历史不同——**不要自作主张加 `--force`**，让用户决定。
- 令牌由命令内部取用，不会出现在命令行、输出或 `.git/config` 里。不要用 `git remote add` 手工拼带令牌的地址，那会把凭据写进仓库配置。
- 不要把推送结果说得比实际更完整：只推了当前分支就不要说"整个项目已经传上去了"。
