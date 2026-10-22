---
name: zhiji-doc
description: 检索、阅读和撰写团队文档（GitLab Wiki）。当用户说「查一下文档」「团队文档里怎么写的」「把这个写成文档」「搜一下 Wiki」，或需要查阅/沉淀团队知识时使用。
---

# 智迹团队文档

用 `zhiji doc` 读写团队 GitLab 的 Wiki。智迹代为保管每人的 GitLab 令牌，读写都以**你本人的身份**进行，权限由 GitLab 判定。无需项目 `init`。不要索要或回显任何令牌。

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

## 查文档

```bash
zhiji doc search "增量上报 游标"
zhiji doc search "部署" --project root/zhiji --limit 10
```

**检索按项目逐个进行。** 命令会打印"搜索了 N 个项目"，无法搜索的项目连同原因打到标准错误。
**搜不到和没搜成是两回事**：看到"N 个项目未能搜索"时要如实告诉用户哪些没覆盖，不能把半次扫描说成"团队文档里没有"。

结果每条包含项目路径、文件路径、命中行号和正文片段。片段通常够判断了；需要全文再取：

```bash
zhiji doc list --project root/zhiji
zhiji doc get  --project root/zhiji --page 接入说明
```

`--project` 可填数字 ID 或完整路径（如 `root/zhiji`）。名称不唯一时命令会列出候选并报错——**此时要把候选呈现给用户让其选择，不要自己挑一个**。

## 写文档

正文从标准输入或 `--file` 读入，必须是 UTF-8 文本，单次最多 1 MB：

```bash
zhiji doc push --project root/zhiji --title "发布流程" <<'EOF'
# 发布流程

## 前置检查
...
EOF
```

覆盖已有页面要带 `--slug <页面标识>`（标识来自 `doc list` 或 `doc search` 的输出）。**不带 `--slug` 是新建**；想更新却漏了它会多出一篇重复文档。

写之前先 `doc search` 或 `doc list` 确认同主题文档是否已存在，存在就更新而不是另起一篇。写入会以用户本人的身份记录在 GitLab 的提交历史里。

## 边界

- 没有绑定 GitLab 时命令返回"尚未绑定"，让用户找管理员在网页「GitLab 账号」页同步一次；令牌过期同理。不要试图自己签发令牌。
- 报 403 说明该项目不允许本人写入。如实说明，不要换个项目写进去。
- 二进制文件不支持，提示用户直接用 git。
- 不要把自己生成的内容说成是从团队文档里查到的；没查到就说没查到。
