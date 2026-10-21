# 智迹生图

通过 `zhiji service image` 使用团队图片模型，支持在服务目录中选择 `gpt-image-2.5` 等已配置模型。Agent 整理提示词，智迹后端代理调用，CLI 保存结果。

需要智迹 CLI 2.2.0 或更新版本与账号登录，无需初始化项目。技能会检查 CLI，缺少时从智迹实例的 `/skills/zhiji-setup/bin/` 下载；不需要上游 API Key。

```bash
zhiji service list
zhiji service image --model gpt-image-2.5 --input "清晨山间湖泊，薄雾" --out ./results/image-job/ --request-id <请求UUID>
```

模型与尺寸以实际服务配置为准。命令返回调用 ID 和图片路径；在网页「调用记录」查看输入、状态、错误、结果文件及上游实际返回的用量。恢复中断请求使用相同请求 ID 与参数，避免重复生成。

安装后用 `$zhiji-image` 或「使用智迹生图」触发。完整工作流见 `skills/zhiji-image/SKILL.md`；本站分发时文件位于 `/skills/zhiji-image/SKILL.md`。

## 实例迁移

团队生产地址已迁移到 `http://172.16.6.163:8090`。保存的旧地址为
`http://172.16.6.139:8089` 时，先为当前终端设置
`ZHIJI_API_URL=http://172.16.6.163:8090`，再运行 `zhiji upgrade`
和 `zhiji skills update`。升级不会自动修改账号保存的地址；永久迁移时先备份
`~/.zhiji/config.json`，仅更新 `apiUrl`，保留原 PAT、项目 ID 和游标。
其它自定义实例保持原样。登录显式使用 `--url <实例地址>`。
插件安装副本通过宿主插件市场更新，CLI 只更新项目内副本；更新后重新读取技能。
