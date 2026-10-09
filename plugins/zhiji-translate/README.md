# 智迹翻译

通过 `zhiji service translate` 调用团队翻译模型，支持短文本、UTF-8 文件与标准输入。Agent 从需求确定目标语言，CLI 保存实际译文。

需要智迹 CLI 2.2.0 或更新版本与账号登录，无需初始化项目。技能会检查 CLI，缺少时从智迹实例的 `/skills/zhiji-setup/bin/` 下载；不需要上游 API Key。

```bash
zhiji service list
zhiji service translate --input-file ./source.txt --target-language English --out ./results/translation-job/ --request-id <请求UUID>
```

模型与语言参数以实际服务配置为准。输入最多 200000 字节。命令返回译文、调用 ID 与文件路径，在网页「调用记录」查看源文、参数、结果和真实用量。恢复中断请求使用相同请求 ID 与参数，避免重复调用。

安装后用 `$zhiji-translate` 或「使用智迹翻译」触发。完整工作流见 `skills/zhiji-translate/SKILL.md`；本站分发时文件位于 `/skills/zhiji-translate/SKILL.md`。
