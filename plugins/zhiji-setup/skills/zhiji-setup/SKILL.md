---
name: zhiji-setup
description: 一条龙把当前目录接入智迹：创建项目、签发 MCP 令牌、写好 Agent 配置、装好上报技能。当用户说「创建 xx 项目」「接入智迹」「配置智迹 MCP」「把这个项目连到智迹」时使用。
---

# 智迹一键接入

本技能调用 `zhiji` 命令行工具。它会按顺序完成四件事，并且可以重复运行：

1. 在智迹创建项目（同标识的项目已存在就复用）；
2. 为本机签发一个 MCP 令牌（同备注的令牌已存在就复用）；
3. 写好本机的 Agent 配置（Claude Code 的 `.mcp.json`，或 Codex 的 `~/.codex/config.toml`）；
4. 把 `zhiji-work-report` 技能装进项目。

下文的 `<技能目录>` 指本 SKILL.md 所在的目录。

## 第一步：确认 CLI 在不在

先看 `<技能目录>/bin/` 下有没有 `zhiji`（Windows 是 `zhiji.exe`）。有就直接跳到下一步。

没有的话需要下载一次。智迹地址默认是 `http://172.16.6.139:8089`，
如果用户明确说了别的实例就用他说的（`~/.zhiji/config.json` 里的 `apiUrl` 也是线索）。

按用户的系统选对应文件，下载后统一改名成 `bin/zhiji`（Windows 为 `bin/zhiji.exe`）：

| 系统 | 文件名 |
| --- | --- |
| Windows | `zhiji-windows-amd64.exe` |
| macOS（Apple 芯片） | `zhiji-darwin-arm64` |
| macOS（Intel） | `zhiji-darwin-amd64` |
| Linux x86_64 | `zhiji-linux-amd64` |
| Linux arm64 | `zhiji-linux-arm64` |

macOS / Linux 下载完必须 `chmod +x`——HTTP 下载不带执行位，漏了会报 Permission denied。

macOS / Linux：

```bash
BASE=http://172.16.6.139:8089; DIR=<技能目录>
mkdir -p "$DIR/bin" \
  && curl -fsSL "$BASE/skills/zhiji-setup/bin/zhiji-$(uname -s | tr 'A-Z' 'a-z')-$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')" -o "$DIR/bin/zhiji" \
  && chmod +x "$DIR/bin/zhiji"
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force <技能目录>\bin | Out-Null
Invoke-WebRequest http://172.16.6.139:8089/skills/zhiji-setup/bin/zhiji-windows-amd64.exe -OutFile <技能目录>\bin\zhiji.exe
```

## 第二步：执行接入

- 类 Unix：`<技能目录>/bin/zhiji init "<项目名>" --dir <项目根绝对路径>`
- Windows：`<技能目录>\bin\zhiji.exe init "<项目名>" --dir <项目根绝对路径>`

`--dir` 传当前项目的根目录（一般是 Git 仓库根）。用户没说项目名时，用目录名，不要自己编一个。

宿主是 Codex 时加 `--agent codex`；两个都要配时用 `--agent both`。默认是 `claude`。

命令会逐行打印做了什么。把结果如实转述给用户，**不要**把输出里的令牌明文复述出来。

## 未登录时

CLI 报「尚未登录」时，让用户自己完成登录，不要替他索要或粘贴令牌：

> 请到智迹「账号设置 → 个人访问令牌」签发一个令牌，然后运行：
> `<技能目录>/bin/zhiji login --token <你的令牌>`
>
> （地址默认 `http://172.16.6.139:8089`，别的实例才要加 `--url`。）

令牌等于整个账号，只在签发时显示一次。不要把它写进任何项目文件、聊天记录或提交信息。

## 不要自己动手

不要自己去写 `.mcp.json`、改 `~/.codex/config.toml`、或手动下载 `zhiji-work-report`。
这些文件里通常已经有开发者自己的其他配置，CLI 会做合并；手写容易整个覆盖掉。

## 收尾

执行完提示用户：

- 重启 Claude Code / Codex，MCP 才会生效；
- 之后用 `$zhiji-work-report` 触发一次上报来验证；
- `<技能目录>/bin/zhiji status --dir <项目根>` 可以检查各项是否就位。
