---
name: zhiji-sfx
description: 通过智迹团队服务按文字描述生成音效，指定时长，保存音频并查询调用记录。用户要求智迹音效或使用团队音效模型时使用。
---

# 智迹音效

用 `zhiji service sfx` 调用智迹后端，后端连接团队 New API，音频由 CLI 保存到本机。无需项目 `init`。不要使用上游账号密码或 API Key。

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

先找 PATH 中的 `zhiji` 或本 SKILL.md 所在目录的 `bin/zhiji`（Windows 为 `bin/zhiji.exe`），运行 `version`。需要 **2.2.0 或更新版本**；版本过低或不存在时，下载到本技能的 `bin/`，不要覆盖系统中的旧文件。

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

## 生成音效

1. 执行 `zhiji service list`，查看 `sfx` 的启用状态、可用模型与默认模型。可能配置 `eleven_sfx`；音效路径和模型须由管理员配置并联调，以实际返回为准，不宣称所有模型已验证。服务未启用时说明需要配置，不尝试绕过智迹直连上游。
2. 把需求整理成声音来源、环境、远近、变化过程与时长；不要把需要配音的台词当作音效描述。生成一个 UUID 作为请求 ID，记下来，并选一个新的输出目录。
3. 发起一次调用，例如：

```bash
zhiji service sfx --input "夜晚林间细雨，近处雨滴敲打树叶，远处轻微雷声" --duration 8 --out ./results/sfx-job/ --request-id <请求UUID>
```

选模型用 `--model <目录中的模型>`。`eleven_sfx` 通过已配置的兼容桥调用，支持 MP3，时长为自动或 0.5–30 秒；不要改为其他格式。其他模型时长与格式参数以实际支持为准。长描述可用 UTF-8 文件 `--input-file <文件>`；`--input-file -` 从标准输入读取。输入最多 200000 字节；`--input`、`--input-file` 和标准输入三选一。

CLI 等待调用完成，成功后打印调用 ID 与音频绝对路径。超时、网络中断时先 `zhiji service show <调用ID>`；如提交响应丢失，使用**相同请求 ID、相同输入与参数**恢复原调用。不要自动更换请求 ID再次生成。失败或中断记录不会在原请求下重新调用上游；只有用户要求重新生成时才创建新请求。

## 结果与记录

提供实际音效、绝对文件路径、模型和调用 ID。在支持本地媒体的宿主用 `![音效](/绝对路径/音效.mp3)` 播放。不要声称听过未实际播放检查的音频；错误如实转述。

`zhiji service calls --capability sfx` 查询历史；`zhiji service show <调用ID> --out ./results/recovered-sfx/` 下载已有音效，不会新建生成调用，已有文件不覆盖。网页「调用记录」可查看描述、时长、状态、错误、真实上游用量，播放与下载音频。普通成员查看本人记录，管理员可以查看全员；上游没返回用量时不要估算或编造费用。
