# 智迹语音合成

通过 `zhiji service speech` 把文案合成为配音，按实际模型能力选择声音和音频格式。模型可包含 `cosyvoice3`、`eleven_tts`，以服务目录为准。

需要智迹 CLI 2.2.0 或更新版本与账号登录，无需初始化项目。技能会检查 CLI，缺少时从智迹实例的 `/skills/zhiji-setup/bin/` 下载；不需要上游 API Key。

```bash
zhiji service list
zhiji service speech --input-file ./narration.txt --out ./results/speech-job/ --request-id <请求UUID>
```

命令返回调用 ID 和音频绝对路径。在网页「调用记录」查看台词、参数、状态与错误，播放、下载实际结果。恢复中断请求使用相同请求 ID 与参数，避免重复合成。

安装后用 `$zhiji-speech` 或「使用智迹配音」触发。完整工作流见 `skills/zhiji-speech/SKILL.md`；本站分发时文件位于 `/skills/zhiji-speech/SKILL.md`。
