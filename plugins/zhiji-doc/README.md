# 智迹共享知识库

`zhiji doc` 检索、阅读和上传智迹自己的共享知识，与 GitLab Wiki 独立。需要 CLI 3.4.0 或更新版本及智迹账号登录，不要求 GitLab 账号或项目初始化。

```bash
zhiji doc categories
zhiji doc search "过审" --category "Android/过审"
zhiji doc get <知识ID>
zhiji doc push --category "iOS/过审" --title "审核经验" --file ./notes.md
```

上传后直接对全员可见。详情与附件索引限制见 `skills/zhiji-doc/SKILL.md`。
