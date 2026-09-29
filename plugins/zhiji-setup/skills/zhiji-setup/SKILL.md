---
name: zhiji-setup
description: 一条龙把当前目录接入智迹：创建项目、签发 MCP 令牌、写好 Agent 配置、装好上报技能。当用户说「创建 xx 项目」「接入智迹」「配置智迹 MCP」「把这个项目连到智迹」时使用。
---

# 智迹一键接入

本技能调用 `zhiji` 命令行工具。它会按顺序完成四件事，并且可以重复运行：

1. 在智迹创建项目，带上你根据这个仓库写的一句说明（同标识的项目已存在就复用）；
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

## 第二步：看一眼项目，写一句说明

这句说明会显示在智迹的项目列表里，让别人一眼知道这是什么项目。**先看再写**，材料按这个顺序找：

1. `README.md` 开头几行——通常直接就是一句话简介；
2. `package.json` / `go.mod` / `pyproject.toml` / `Cargo.toml` 的 `description` 字段和依赖；
3. 顶层目录结构（有 `apps/`？`src/` 下是什么？）。

写成**一句中文**，30–60 字，说清**这是什么、给谁用、什么技术栈**。举例：

- `面向研发团队的 AI Agent 工作观测平台，Go API + React 控制台`
- `微信公众号内容自动生成与发布工具，Python + FastAPI`

几条要求：

- **看不出来就别写**，直接省掉 `--description`。留空比写一句错的强——错的说明会被当成事实读。
- 不要写成「这是一个项目」「一个 Go 项目」这种等于没说的话。
- 不要把 README 整段抄进去，一句话。

## 第三步：执行接入

- 类 Unix：`<技能目录>/bin/zhiji init "<项目名>" --dir <项目根绝对路径> --description "<说明>"`
- Windows：`<技能目录>\bin\zhiji.exe init "<项目名>" --dir <项目根绝对路径> --description "<说明>"`

`--dir` 传当前项目的根目录（一般是 Git 仓库根）。用户没说项目名时，用目录名，不要自己编一个。

宿主是 Codex 时加 `--agent codex`；两个都要配时用 `--agent both`。默认是 `claude`。

说明只在项目**还没有说明**时写入：已经有的（可能是别人在网页上精心写的）不会被覆盖，
重复执行也不会反复改写。想改已有的说明，去网页「项目管理」里编辑。

命令会逐行打印做了什么。把结果如实转述给用户，**不要**把输出里的令牌明文复述出来。

## 未登录时

CLI 报「尚未登录」时，让用户自己完成登录，不要替他索要或粘贴令牌：

> 请到智迹「令牌管理」签发一个令牌，然后运行：
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
