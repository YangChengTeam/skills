# zhiji-setup

一条龙把当前目录接入智迹：创建项目、签发 MCP 令牌、写好 Agent 配置、装好上报技能。

说一句「创建 xx 项目」就够了，技能会调用 `zhiji` 命令行工具依次完成四件事：

1. 在智迹创建项目——同标识的项目已存在就复用；
2. 为本机签发一个 MCP 令牌——同备注的令牌已存在就复用；
3. 写好本机的 Agent 配置——Claude Code 的 `.mcp.json`，或 Codex 的 `~/.codex/config.toml`；
4. 把 [`zhiji-work-report`](../zhiji-work-report) 装进项目，并把 `.mcp.json` 加进 `.gitignore`。

之后重启 Agent，用 `$zhiji-work-report` 就能上报。

## 安装

```bash
claude plugin marketplace add YangChengTeam/skills
claude plugin install zhiji-setup@yangcheng
```

## CLI 从哪来

本仓库**只放技能文本，不放二进制**——公开仓库塞几十 MB 的可执行文件每次发布都会让 Git
历史永久膨胀。技能首次运行时发现 `bin/zhiji` 不存在，会从你们自己的智迹站点下载对应平台的
那一个（Windows / macOS Intel / macOS Apple 芯片 / Linux x86_64 / Linux arm64）：

```
http://<智迹地址>/skills/zhiji-setup/bin/zhiji-<os>-<arch>
```

macOS 与 Linux 下载完要 `chmod +x`——HTTP 下载不带执行位。

反正连不上智迹站点的话，这个 CLI 本来也没用——它干的每件事都要调智迹的接口。

## 前提

先在智迹「账号设置 → 个人访问令牌」签一个令牌（只显示一次），然后：

```bash
<技能目录>/bin/zhiji login --token <令牌>
```

地址默认团队部署，接别的实例时才加 `--url`。

令牌等于整个账号，只保存在自己机器的 `~/.zhiji/config.json`（0600）。不要写进项目文件或
提交记录。也可以用环境变量 `ZHIJI_API_URL` / `ZHIJI_PAT` 代替 `login`。

## 幂等

`init` 可以重复跑：项目按 slug 复用，令牌按备注复用，`.mcp.json` 与 `config.toml` 是
**合并**而不是覆盖（你已有的其他 MCP 服务不会被动到），`.gitignore` 不会重复追加。

## 排查

```bash
<技能目录>/bin/zhiji status --dir <项目根>
```

会打印服务地址、登录状态，以及 `.mcp.json` 和技能文件是否就位。
