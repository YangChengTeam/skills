---
name: zhiji-code
description: 检索和读取团队各项目的代码，并把文件提交到 GitLab 仓库。当用户说「搜一下代码怎么写的」「别的项目里怎么实现的」「把这个文件提交上去」，或需要跨项目查找实现与惯例时使用。
---

# 智迹团队代码

用 `zhiji code` 搜索、阅读和提交团队 GitLab 仓库里的文件。智迹代为保管每人的 GitLab 令牌，读写都以**你本人的身份**进行，权限由 GitLab 判定。无需项目 `init`。不要索要或回显任何令牌。

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

先找 PATH 中的 `zhiji` 或本 SKILL.md 所在目录的 `bin/zhiji`（Windows 为 `bin/zhiji.exe`），运行 `version`。需要 **2.7.0 或更新版本**；版本过低或不存在时，下载到本技能的 `bin/`，不要覆盖系统中的旧文件。

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

## 搜代码

```bash
zhiji code projects
zhiji code search "RedisQueue"
zhiji code search "func NewServiceGateway" --project root/zhiji --limit 10
```

**检索按项目逐个进行。** 命令会打印"搜索了 N 个项目"，无法搜索的项目连同原因打到标准错误。
**搜不到和没搜成是两回事**：有项目没搜成时要如实说明覆盖范围，不能把半次扫描说成"团队代码里没有这种写法"。

这是基础搜索（相当于 `git grep`），不是语义检索：关键词要用实际会出现在代码里的标识符，不要用自然语言描述。结果带文件路径和命中行号，看全文用：

```bash
zhiji code get --project root/zhiji --path server/internal/app/service_gateway.go
zhiji code get --project 1 --path src/main.go --ref release-2.0 --out ./main.go
```

`--project` 可填数字 ID 或完整路径。名称不唯一时命令会列出候选并报错——**把候选呈现给用户让其选择，不要自己挑一个**。`--out` 不会覆盖已有文件。

## 提交代码

```bash
zhiji code push --project root/zhiji --path docs/deploy.md -m "补充部署说明" --file ./deploy.md
```

内容来自 `--file` 或标准输入，必须是 UTF-8 文本，单次最多 1 MB。省略 `--branch` 时提交到项目默认分支。命令会自己判断该用新建还是更新，不需要你指定。

提交前务必做到：

1. **先 `code get` 看现有内容**，在此基础上改，不要用凭空重写的版本覆盖别人的工作。
2. 提交说明写清楚改了什么，这条会进入 Git 历史。
3. 确认目标分支是对的。**这是直接写入远端分支，没有 code review 环节**，不要用它推送未经用户确认的改动。

提交作者记为用户本人。成功后输出提交短 ID、分支和链接，把链接给用户。

## 边界

- 没有绑定 GitLab 时命令返回"尚未绑定"，让用户找管理员在网页「GitLab 账号」页同步一次；令牌过期同理。不要试图自己签发令牌。
- 报 403 说明该项目不允许本人写入。如实说明，不要改推到别的项目。
- 二进制文件、批量改动、重命名和删除不走这里，提示用户直接用 git。
- 这个命令适合单文件的小改动和文档补充。成规模的修改应该在本地用 git 正常提交和评审。
- 不要把自己写的代码说成是从团队仓库里查到的惯例；没查到就说没查到。
