# 智迹音效

通过 `zhiji service sfx` 按场景描述生成音效，支持传入时长并保存实际音频。模型、音效请求路径与参数须由管理员配置并联调，以服务目录的启用状态为准。

需要智迹 CLI 2.2.0 或更新版本与账号登录，无需初始化项目。技能会检查 CLI，缺少时从智迹实例的 `/skills/zhiji-setup/bin/` 下载；不需要上游 API Key。

```bash
zhiji service list
zhiji service sfx --input "林间雨滴敲打树叶，远处轻微雷声" --duration 8 --out ./results/sfx-job/ --request-id <请求UUID>
```

命令返回调用 ID 和音频绝对路径。在网页「调用记录」查看提示词、时长、状态与错误，播放、下载实际结果。恢复中断请求使用相同请求 ID 与参数，避免重复生成。

安装后用 `$zhiji-sfx` 或「使用智迹生成音效」触发。完整工作流见 `skills/zhiji-sfx/SKILL.md`；本站分发时文件位于 `/skills/zhiji-sfx/SKILL.md`。
