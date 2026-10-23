---
name: zhiji-doc
description: 检索、阅读和上传智迹共享知识库。当用户说「查知识库」「团队有没有这方面经验」「把经验上传到知识库」，或 Agent 需要团队共享知识时使用。
---

# 智迹共享知识库

`zhiji doc` 只连接智迹共享知识库，与 GitLab Wiki、GitLab 账号和项目代码库无关。所有已登录用户可读取和上传；管理员在网页「开发者服务 → 知识库」建立一级与二级分类。Android / 过审和 iOS / 过审是两条不同的分类路径。无需 `zhiji init`。

## 准备

先运行 `zhiji version`，需要 **3.4.0 或更新版本**。版本较旧时用 `zhiji upgrade` 更新；没有 CLI 时按 `zhiji-setup` 的安装说明获取对应平台的二进制。若尚未登录，让用户在智迹网页「令牌管理」签发个人访问令牌并自行执行 `zhiji login`。不要索要或回显令牌。

团队实例为 `http://172.16.6.163:8090`。若当前账号配置仍指向旧实例 `http://172.16.6.139:8089`，本次命令设置 `ZHIJI_API_URL=http://172.16.6.163:8090`；用户明确指定其它实例时按其要求。不要把整个账号配置或 `.env.zhiji` 输出到对话中。

## 检索和阅读

```bash
zhiji doc categories
zhiji doc search "应用商店过审" --category "Android/过审"
zhiji doc list --category "iOS/过审"
zhiji doc get <知识ID>
zhiji doc download <知识ID> <附件ID> --out ./reference.pdf
```

先搜索，再用稳定的知识 ID 读取全文。回答时注明条目标题、分类和 ID；不能把自己推测的内容说成知识库记录。分类必须用完整路径或 ID；没有匹配时不要猜一个相近分类。

正文与可提取的附件文本会在 `get` 中显示。附件显示「未建立全文索引」时，搜索没有覆盖它的全文；需要时下载原文件查看，不能把这次检索说成覆盖了所有附件。GitLab 项目 Wiki 仍在各项目网页的「代码库」页签，不通过 `zhiji doc` 查询。

## 上传

上传前先查同主题内容，确认所选二级分类正确。新知识上传后立即对全员可见。

```bash
zhiji doc push --category "Android/过审" --title "应用商店审核经验" --file ./review.md
zhiji doc push --category "AI/提示词" --title "提示词规范" < ./prompt.md
```

`--file` 将原文件作为附件上传；省略时从标准输入读取 Markdown 正文。支持 md、txt、csv、json、pdf、docx 和常见图片，单文件最多 20 MB。PDF 若无法提取文字、图片没有 OCR 时，文件仍可下载，但不会参与全文搜索。上传是写入全员共享知识，应先核对内容和分类。

旧 `zhiji doc --project/--page/--slug` 参数属于已退出的 GitLab Wiki CLI，用在新版知识库命令会报错。不要尝试改用 GitLab 仓库代替知识库分类。
