# zhiji-setup

创建项目、安装技能，并按项目类型从全库下发相关要求文档。

说一句「创建 xx 项目」，技能会调用智迹 CLI 依次完成：

1. 创建项目并填写一句仓库说明，同标识的项目已存在就复用；
2. 写入只含项目 ID 的 `.env.zhiji`，账号令牌留在本机；
3. 安装 [`zhiji-work-report`](../zhiji-work-report) 和 [`zhiji-ticket`](../zhiji-ticket)。
4. 按项目名称、类型和关键词搜索当前账号可见的全部 GitLab 项目，下载相关文档原文与来源到 `.zhiji/documents/`，加入 Agent 入口索引提示。

`init` 可以重复运行，会保留已有配置和说明，并清理旧版本留下的智迹 MCP 配置。

## 实例迁移

团队生产地址已迁移到 `http://172.16.6.163:8090`。保存的旧地址为
`http://172.16.6.139:8089` 时，先为当前终端设置
`ZHIJI_API_URL=http://172.16.6.163:8090`，再运行 `zhiji upgrade`
和 `zhiji skills update`。升级不会自动修改账号保存的地址；永久迁移时先备份
`~/.zhiji/config.json`，仅更新 `apiUrl`，保留原 PAT、项目 ID 和游标。
其它自定义实例保持原样。登录显式使用 `--url <实例地址>`。
插件安装副本通过宿主插件市场更新，CLI 只更新项目内副本；更新后重新读取技能。

## 安装

```bash
claude plugin marketplace add YangChengTeam/skills
claude plugin install zhiji-setup@yangcheng
```

Codex 使用：

```bash
codex plugin marketplace add YangChengTeam/skills
codex plugin add zhiji-setup@yangcheng
```

## CLI 与登录

需要智迹 CLI **2.9.0 或更新版本**。技能会检查版本，缺少或版本过低时从智迹实例下载到本技能的 `bin/`。支持 Windows、macOS Intel / Apple 芯片、Linux x86_64 / arm64。

APK 新项目可显式传 `--type apk`；已有工程按明确文件识别类型，重复初始化保留服务端设置。
`--keywords "安全规范,发布要求"` 补充检索词，已有项目用 `zhiji doc related` 刷新。
接入后读取资料索引和适用原文；匹配不是自动强制要求，未匹配、部分失败或截断会明确提示。
人工修改的资料保留，不在项目写入凭据。网页「文档与代码 → 项目相关」可检索及调整配置。

2.5.0 取消 400 文件扫描限制，支持归档/压缩会话与 `report --history` 幂等补采。
2.6.0 补采改为持久化分批计划，每批分别审阅提交，中断后同一命令继续；
`report --history --status` 查看进度，`--history --new` 开新计划。
项目技能可用 `zhiji skills update --auto on` 首次更新并接管，之后 report 前每日检查；
新 init 安装的副本默认受管。自动更新保留人工修改，手动更新先备份，不改写宿主插件缓存。

2.3.0 起可运行 `<技能目录>/bin/zhiji upgrade --check` 查看新版本，运行同一路径的
`zhiji upgrade` 下载并校验后升级。升级保留旧版备份，不改账号配置或待提交批次。
2.2.x 需要手动重新下载一次以获得升级命令。显式 `--dir` 严格限定项目范围，
省略时寻找最近的项目配置或 Git 边界，不允许整盘采集。

本仓库只分发技能文本，二进制由智迹实例提供：

```text
http://<智迹地址>/skills/zhiji-setup/bin/zhiji-<os>-<arch>
```

macOS / Linux 下载后需要 `chmod +x`。默认实例是 `http://172.16.6.163:8090`，用户指定的实例优先。

先到智迹「令牌管理」签发个人访问令牌，再在本机运行：

```bash
<技能目录>/bin/zhiji login --url http://172.16.6.163:8090 --token <你的令牌>
```

其他实例加 `--url <实例地址>`。账号令牌只保存在 `~/.zhiji/config.json`，不要写进项目文件或提交记录。

## 使用与排查

安装后用 `$zhiji-setup` 或「把这个项目接入智迹」触发。完整工作流见 [SKILL.md](skills/zhiji-setup/SKILL.md)。

接入完成后只用 status 验证配置，不为安装、登录、接入或验证生成工作日志。有业务工作后
再用 `$zhiji-work-report` 上报，纯接入批次由该技能使用 `skip` 过滤。
用 `$zhiji-ticket` 提工单；账号级生图、翻译、语音、音效服务由独立技能提供，无需初始化项目。

```bash
<技能目录>/bin/zhiji status --dir <项目根>
```
