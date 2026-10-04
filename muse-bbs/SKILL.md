---
name: muse-bbs
description: 在 Muse AI 交流社区论坛（https://bbs.luyuanlab.eu.org）发帖、回答、评论。当用户想往论坛发内容时使用，比如"帮我发个帖子"、"去论坛问个问题"、"帮我回复那个帖子"。
---

# Muse AI 论坛发帖 Skill

通过 API 在 Muse AI 交流社区（Apache Answer 搭建）发帖、回答、评论。

论坛地址：`https://bbs.luyuanlab.eu.org`
API 基地址：`https://bbs.luyuanlab.eu.org`

## 身份验证

发帖需要论坛账号。用户先在 https://bbs.luyuanlab.eu.org 注册（只需用户名+密码，无需邮箱验证）。

登录拿 token（token 当天有效，同一会话内复用，不要重复登录）：

```bash
curl -s -X POST https://bbs.luyuanlab.eu.org/answer/api/v1/user/login/email \
  -H "Content-Type: application/json" \
  -d '{"e_mail":"用户邮箱","pass":"用户密码"}'
```

返回 `data.access_token`。后续请求带请求头 `Authorization: <access_token>`（注意：直接放 token，不是 `Bearer` 前缀）。

**安全规则**：只在用户主动提供时使用邮箱密码；拿到 token 后不再需要密码；不要把密码写入任何文件。

如果用户没给账号信息，先问他要（邮箱+密码），或引导他先去论坛注册。

## 发帖（提问）

```bash
curl -s -X POST https://bbs.luyuanlab.eu.org/answer/api/v1/question \
  -H "Content-Type: application/json" \
  -H "Authorization: <access_token>" \
  -d '{
    "title": "帖子标题（6-150字）",
    "content": "正文，支持 Markdown",
    "tags": [{"slug_name": "tag-slug", "display_name": "显示名"}]
  }'
```

- `title` 必填，6~150 个字符
- `content` 支持 Markdown，65535 字以内
- `tags` 至少给 1 个。常用标签 slug：`cloudflare`、`tutorial`（教程）、`deploy`（部署）；也可以按内容新建，比如 `{"slug_name": "muse-ai", "display_name": "Muse AI"}`
- 成功返回 `data.id` 即问题 ID，帖子链接为 `https://bbs.luyuanlab.eu.org/questions/<id>`

## 回答问题

```bash
curl -s -X POST https://bbs.luyuanlab.eu.org/answer/api/v1/answer \
  -H "Content-Type: application/json" \
  -H "Authorization: <access_token>" \
  -d '{"question_id": "问题ID", "content": "回答正文（Markdown，至少6字）"}'
```

## 发评论

```bash
curl -s -X POST https://bbs.luyuanlab.eu.org/answer/api/v1/comment \
  -H "Content-Type: application/json" \
  -H "Authorization: <access_token>" \
  -d '{"object_id": "问题或回答的ID", "original_text": "评论内容（2-600字）"}'
```

## 工作流程

1. 用户说"发个帖子"但没给标题内容 → 问他要标题和正文要点
2. 用户给了完整内容 → 直接发，**发之前把标题+正文+标签展示给他确认**（首次使用时）；确认后调用 API
3. 发成功后把帖子链接给他
4. 如果用户想匿名/免登录发帖：论坛不支持，只能用他自己的账号发

## 注意事项

- 内容要和 AI/技术交流相关，别发广告和垃圾内容
- 中文社区，正文用简体中文
- 发帖频率别太高，避免被限流
